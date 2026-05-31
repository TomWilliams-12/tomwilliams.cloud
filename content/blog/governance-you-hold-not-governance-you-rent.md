---
title: "Governance You Hold, Not Governance You Rent — A Stratum Case Study"
date: 2026-05-31
draft: false
tags: ["aws", "governance", "multi-account", "security", "soc2", "case-study"]
---

Most AWS governance tools ask you to do something quietly radical: hand a third party a role into your Organization and let your IAM policies, CloudTrail events, and resource inventory flow out to someone else's SaaS backend. It works. It also tends to be the exact thing that stalls in procurement for three months while security asks where the data goes.

I build the other kind. **Stratum** is a self-hosted AWS governance platform that deploys *into* the customer's own AWS Organization, scans every member account, and produces prioritised, deduplicated findings with concrete remediation paths — without the data plane ever leaving the customer's AWS boundary.

This is a case study, not a product page. I'm writing it up because Stratum is how I operationalise governance work for clients, and it's a fair proxy for how I approach the job generally: primitives first, append-only audit trails, decisions written down, and no magic.

## The problem: multi-account estates rot

If you run a growing AWS estate, you already know the failure mode. It isn't one big breach. It's slow decay:

- **Drift.** A landing zone that was clean at launch accumulates one-off exceptions, hand-edited security groups, and "temporary" public buckets.
- **Alert fatigue.** Five tools each emit findings in their own schema. Nobody triages all five, so nothing gets triaged.
- **Orphaned findings.** A finding gets noticed, half-fixed, and lost. Six months later it resurfaces in an audit.
- **Evidence scrambles.** SOC 2 season arrives and three engineers spend a week screenshotting consoles to prove controls were operating.

The common thread is that governance is treated as a periodic event rather than a running system. Stratum's job is to make it a running system — one a human can actually keep up with.

## What Stratum is

A scanner-plus-frontend platform that lives entirely in the customer's AWS Organization. Scanners run on a schedule, enumerate accounts through the Organizations API, assume a read-only cross-account role in each member account, and emit findings into one shared schema. A frontend reads those findings through a versioned API and renders a single, scope-parameterised view: whole-org, per-account, or per-module.

The headline differentiator is boundary, not features:

> Nothing that touches real AWS data leaves the customer's account. Scanners, the findings store, and the evidence bucket all live in the customer's tooling account.

That single property is what gets a tool like this through a security review instead of dying in it. You're not auditing a vendor's data-handling claims — there's no data leaving to handle.

## Coverage that lands in one queue

Breadth only helps if it converges. Stratum runs around **eleven independent scanner modules**:

- IAM posture
- Compute (EC2 / EBS)
- Networking (VPC, security groups, flow logs, Transit Gateway)
- S3
- RDS
- DynamoDB
- Lambda
- AWS Config
- SSM patch coverage
- Cost
- …plus a SOC 2 compliance-mapping layer that sits on top

Every module writes to the **same findings schema**, so the output is one deduplicated, prioritised queue a human can triage — not eleven dashboards in eleven dialects. Modules don't import from each other; where cross-module context matters (say, an internet-exposed Lambda or RDS instance), module-local correlators enrich a finding by reading the shared open-findings index. There's deliberately no shared aggregation god-layer to rot.

## Architecture worth showing

A few choices that reflect how I like to build:

**Terraform manages CloudFormation StackSets; StackSets deploy the modules.** Each module is independently deployable. One module failing to roll out never takes the others down with it — you get partial coverage and a clear signal, not an all-or-nothing apply.

**A server-side read model keeps triage fast at scale.** The system is designed for large estates producing tens of thousands of findings. The frontend never talks to the database directly; it calls a versioned API with an explicit contract. That decoupling is also why the platform can later lift its control plane out of the customer account without a rewrite — the data plane stays put either way.

**One apply per customer.** Root configs compose modules and own their own backend and providers; a single `modules = { ... }` toggle decides which scanners are enabled. Turning governance on for a new account is configuration, not a project.

## Trust and safety by design

A platform with org-wide reach has to be conservative about reach. Stratum is **read-only by default** — the cross-account scanner role carries a permission boundary that blocks every write action, org-wide, full stop. The default deployment cannot change your infrastructure even if it wanted to.

Remediation is designed as a deliberate, opt-in path rather than a default. When it's enabled — module by module, account by account — an action runs as an SSM Automation document under a *separate*, tightly-scoped per-module role whose permission boundary denies destructive operations (no touching `admin`/`root` roles, no `*:*` wildcards). Every action is logged immutably. The posture is: prove you trust a module before you ever let it write, and even then it writes inside a box you defined.

## Findings you can put in front of an auditor

Findings are **immutable and append-only**. Each finding is written once; its status (open, suppressed, resolved) is *computed* from an append-only event stream of status, note, and assignee changes — not by mutating a row. Evidence lands in an S3 bucket with Object Lock and long retention. The result is a tamper-evident history: you can show not just the current state, but exactly when something was detected, acknowledged, and closed.

The SOC 2 layer maps technical findings to Trust Services Criteria — **honestly**. It evidences control state and detection. It does not print a green "you are 94% compliant" number, because that number is fiction and auditors know it. It tells you which controls have supporting evidence and which don't. That honesty is the point; a compliance layer that flatters you is worse than none.

## Why this is the case study

Stratum isn't the deliverable I'm selling — it's proof of how I work. The same instincts that shaped it are the ones I bring to client engagements:

- **Primitives first.** StackSets, SSM, DynamoDB, Object Lock — well-understood AWS building blocks composed deliberately, not a tower of bespoke abstractions.
- **Append-only audit trails.** If it matters, it's reconstructable from an event stream.
- **Decisions written down.** Architecture decisions are recorded so the *why* survives the people.
- **No overclaiming.** Read-only until you say otherwise; honest compliance mapping; partial coverage surfaced rather than hidden.

If your multi-account estate is growing faster than your ability to govern it — drift, alert fatigue, audit-time scrambles — that's exactly the shape of problem I work on.

## Let's talk

I'm happy to spend 30 minutes, no pitch, looking at your AWS Organizations, IAM, and multi-account posture and telling you where the real risk sits. If that's useful, reach out and we'll find a time.
