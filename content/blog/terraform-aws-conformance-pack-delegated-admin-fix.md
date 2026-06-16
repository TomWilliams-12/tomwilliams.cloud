---
title: "Fixing a 4-Year-Old Terraform Timeout in the AWS Provider"
date: 2026-06-16
draft: false
summary: "Why aws_config_organization_conformance_pack times out from a delegated admin account, and the fix I got merged into the Terraform AWS provider."
tags: ["aws", "terraform", "open-source", "aws-config", "conformance-packs", "multi-account"]
---

Deploying AWS Config conformance packs across an Organization sounds like a solved problem until Terraform sits on `Still creating...` for half an hour and then gives up on a pack that deployed fine five minutes in.

I hit this running Config from a delegated admin account, found a four-year-old GitHub issue full of people hitting the same wall, and ended up getting a fix merged into the [Terraform AWS provider](https://github.com/hashicorp/terraform-provider-aws). Here's what was actually going on.

## The symptom

You apply an organization conformance pack. CloudFormation shows `CREATE_COMPLETE` within a few minutes. The AWS Config console says **Deployment Successful**. Terraform, meanwhile, is still waiting:

```text
aws_config_organization_conformance_pack.cis: Still creating... [37m5s elapsed]
...
Error: error waiting for Config Organization Conformance Pack (cis-conformance-pack) to be created:
timeout while waiting for state to become 'CREATE_SUCCESSFUL'
(last state: 'CREATE_IN_PROGRESS', timeout: 30m0s)
```

Then it gets annoying. Because the create "failed", Terraform taints the resource and tries to destroy and recreate it on the next apply. The pack is deployed and working. Terraform just has no idea.

[Issue #24545](https://github.com/hashicorp/terraform-provider-aws/issues/24545) had been open since 2022 with a steady trickle of "me too" comments. Bumping the create timeout didn't help. Swapping between `template_body` and `template_s3_uri` didn't help. People ended up doing manual uploads or shelling out with `local-exec`.

## What's actually happening

The creation waiter polls `DescribeOrganizationConformancePackStatuses` and blocks until the aggregate status flips to `CREATE_SUCCESSFUL`. From the Organization management account, that works.

From a **delegated administrator** account, which is how most sensible landing zones run Config, it doesn't. The aggregate status never moves off `CREATE_IN_PROGRESS`, even once every member account has finished. The per-account detail is correct the whole time. It's only the rolled-up summary that gets stuck. So the waiter sits there polling a value that's never going to change, and eventually times out.

That's really an AWS API quirk rather than a Terraform bug. But Terraform is where you feel it, so that's where the workaround has to live.

## The fix

There's a second, more granular API the provider was already using elsewhere: `GetOrganizationConformancePackDetailedStatus`. It returns status per member account instead of one aggregate value, and it stays accurate even from a delegated admin account.

So the fix leaves the aggregate poll alone and adds a fallback. When the aggregate status comes back as `CREATE_IN_PROGRESS`, ask the detailed API how many accounts are genuinely still in progress. If the answer is zero, the deployment has finished and we can treat it as successful:

```go
status := output.Status

// The DescribeOrganizationConformancePackStatuses API may not
// transition the aggregate status from CREATE_IN_PROGRESS to
// CREATE_SUCCESSFUL when called from a delegated administrator
// account, even after all member accounts have completed
// deployment. Work around this by checking per-account detailed
// statuses: if no accounts are still in progress, the
// deployment has finished.
if status == types.OrganizationResourceStatusCreateInProgress {
    if v, err := findOrganizationConformancePackDetailedStatusesByTwoPartKey(
        ctx, conn, name, types.OrganizationResourceDetailedStatusCreateInProgress,
    ); err == nil && len(v) == 0 {
        status = types.OrganizationResourceStatusCreateSuccessful
    }
}

if status == types.OrganizationResourceStatusUpdateInProgress {
    if v, err := findOrganizationConformancePackDetailedStatusesByTwoPartKey(
        ctx, conn, name, types.OrganizationResourceDetailedStatusUpdateInProgress,
    ); err == nil && len(v) == 0 {
        status = types.OrganizationResourceStatusUpdateSuccessful
    }
}

return output, string(status), nil
```

The same thing happens on `UPDATE_IN_PROGRESS`, so it gets the same treatment. The issue had reports of `unexpected state 'UPDATE_IN_PROGRESS'` for exactly that reason.

A few things kept this safe rather than reckless:

- It only reaches for the detailed API when the aggregate status is one of the two states known to get stuck. The normal management-account path is untouched.
- If the detailed call errors, it falls back to the original aggregate status instead of guessing. A transient API blip never gets read as "done".
- Querying with a `CREATE_IN_PROGRESS` filter and getting nothing back is a clean signal that every member account has moved on.

## Testing it

This one is genuinely awkward to test, because it only shows up with a real multi-account Organization and a delegated admin configured. That's not something CI has lying around. The PR adds a `delegatedAdministrator` acceptance test built on the provider's alternate-account and organization-member pre-checks to stand that topology up.

Alongside that I verified it by hand against a live Organization, where I confirmed the aggregate status stayed pinned on `CREATE_IN_PROGRESS` while CloudFormation reported `CREATE_COMPLETE` in the delegated admin account, and that with the fix applied the waiter detected completion and returned cleanly.

## What I took from it

The resource was never broken. It deployed every time. What was broken was its view of its own status from one particular calling context, and multi-account AWS is full of that. Plenty of APIs behave slightly differently depending on whether you're calling from the management account, a delegated admin, or a member, and it's worth assuming that's a possibility whenever something works in one account and hangs in another.

The other thing: a four-year-old issue with a wall of frustrated comments looks intimidating, but the actual change was small. A fallback in one status function and a test to cover it. The hard part wasn't the code, it was that nobody had pinned down the root cause yet.

The fix is merged and will ship in an upcoming provider release. If you've been working around this with manual uploads or shell-outs, you can drop them once it lands.

*Links: [PR #47072](https://github.com/hashicorp/terraform-provider-aws/pull/47072) and [Issue #24545](https://github.com/hashicorp/terraform-provider-aws/issues/24545)*
