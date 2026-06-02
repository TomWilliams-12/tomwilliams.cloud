---
title: "Centralizing Firewall in a Multi-Account Org: Network Firewall's New Transit Gateway Attachment"
date: 2026-06-02
draft: false
tags: ["aws", "network-firewall", "transit-gateway", "control-tower", "multi-account", "security"]
---

If you run a multi-account AWS Organization, you've almost certainly built (or inherited) a centralized inspection VPC. It works, but it's fiddly: dedicated firewall subnets, a separate Transit Gateway attachment subnet per AZ, appliance mode, and a small pile of route tables you have to get exactly right or traffic silently stops being inspected.

AWS recently made that whole pattern a lot simpler. Network Firewall now supports **native attachment to Transit Gateway** — you attach the firewall directly to the TGW and skip the inspection VPC plumbing entirely. This post covers what changed, how it maps onto a Control Tower / AFT landing zone, and when it's worth migrating.

## The old model: an inspection VPC you have to babysit

In the classic centralized design, traffic between spoke VPCs (and to/from on-prem) is hairpinned through a dedicated inspection VPC that you own and operate. The moving parts:

- **Two subnets per AZ** in the inspection VPC — one for the TGW attachment, one for the firewall endpoint.
- **Two TGW route tables** — a spoke route table (default route pointing at the inspection VPC attachment) and a firewall route table (spoke routes propagated in so return traffic has a path home).
- **A VPC route table per TGW subnet**, each with a `0.0.0.0/0` default pointing at the firewall endpoint *in the same AZ*.
- **Appliance mode enabled** on the TGW attachment, so the flow hash pins a connection to a single firewall ENI and return traffic comes back symmetrically through the same AZ.

None of this is hard, exactly. But it's a lot of state to keep correct across every account you onboard, and it's the kind of thing that breaks subtly — an asymmetric route, a forgotten propagation — in ways that are annoying to debug.

## The new model: attach the firewall to the TGW directly

With native TGW attachment, you create the firewall and tell it which Transit Gateway to attach to. That's the bulk of it. AWS deploys the firewall endpoints into an **AWS-managed VPC** on your behalf, and from your side the firewall shows up as a Transit Gateway attachment that you route traffic to — like any other attachment.

What disappears:

- No inspection VPC to create or own.
- No firewall/attachment subnets to lay out per AZ.
- No per-subnet route tables pointing at AZ-local endpoints.
- Appliance-mode symmetry is handled as part of the integration rather than something you wire up by hand.

You still control the firewall policy, rule groups, and logging exactly as before. What you stop managing is the *network scaffolding* around the firewall.

## Where this fits in a Control Tower / AFT landing zone

In most landing zones, inspection lives in a dedicated network or security account, with the TGW shared out via RAM and spoke VPCs attached as accounts get vended through AFT. The native attachment model slots into that cleanly:

1. **Network account owns the firewall.** Create the Network Firewall and its TGW attachment in the centralized networking account that already owns the Transit Gateway.
2. **Spoke routing stays the same.** AFT-vended account VPCs attach to the TGW and associate with a spoke route table whose default route points at the firewall attachment. The big win: there's no longer an inspection-VPC attachment to special-case in your account baseline.
3. **Codify it.** Whether you template the firewall in your AFT `global-customizations` or a separate networking pipeline, the resource definition shrinks — you're declaring a firewall and an attachment, not a VPC, subnets, and a route-table matrix.

For new orgs, this means one fewer bespoke component to stand up. For new accounts, onboarding is just "attach to TGW, point default route at the firewall attachment" — no inspection-VPC-aware customization needed.

## Should you migrate an existing inspection VPC?

If you already have a working centralized inspection VPC, there's no fire drill here — the old model still works fine. But native attachment is worth planning a migration toward if any of these resonate:

- You're tired of maintaining the inspection-VPC subnet/route-table boilerplate across AZs.
- Your account baseline carries special-case logic for the inspection attachment that you'd love to delete.
- You're scaling AZs or regions and don't want to hand-replicate the subnet layout each time.

A few things to check before you commit:

- **Region availability** — native TGW support rolled out broadly, but confirm it's live in every region your org operates in.
- **Cutover is a routing change.** You'll be repointing TGW route tables from the old inspection-VPC attachment to the new firewall attachment. Plan it as a controlled flow cutover and watch your firewall logs to confirm traffic is still being inspected (not bypassed) on both directions.
- **Logging and policy parity.** Reuse your existing firewall policy and rule groups so behaviour is identical post-cutover — only the attachment model changes.

## Takeaway

The native Transit Gateway attachment doesn't change *what* Network Firewall does — it removes the inspection-VPC scaffolding you used to have to build and maintain around it. In a multi-account org, that's less per-account routing state, a simpler account baseline, and one fewer thing to get subtly wrong. If you're standing up a new landing zone, start here. If you're running the classic model today, it's worth putting a migration on the roadmap.
