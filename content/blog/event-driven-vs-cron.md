---
title: "Why Event-Driven Infrastructure Beats Cron Jobs"
date: 2026-03-14
summary: "How replacing scheduled scans with EventBridge rules cut our compliance remediation time and simplified our Terraform codebase."
tags: ["aws", "eventbridge", "automation", "terraform"]
---

If you've spent any time managing infrastructure at scale, you've probably written a cron job that polls for something. Maybe it checks for untagged resources every hour, or scans for missing CloudWatch alarms on a schedule. It works. It's simple. And it's almost always the wrong long-term answer.

I recently rebuilt one of these systems — a compliance remediation tool that ensures every EC2 instance in our multi-account AWS organisation has CloudWatch CPU alarms — and the shift from scheduled polling to event-driven architecture made a surprising difference.

## The cron approach

The original setup ran a Lambda on a CloudWatch Events schedule every 30 minutes. It would:

1. Assume a role into each member account
2. List all EC2 instances
3. Check for the existence of CloudWatch alarms
4. Create any that were missing

This worked, but had problems:

- **Latency**: A new instance could run for up to 30 minutes without monitoring
- **Cost**: Every run scanned every instance, even if nothing had changed
- **Complexity**: The Lambda needed to handle pagination across dozens of accounts, manage rate limiting, and deal with partial failures gracefully
- **Noise**: CloudWatch Logs filled up with successful "nothing to do" runs

## The event-driven approach

The replacement uses EventBridge rules deployed to each member account via StackSets. When an EC2 instance launches or has its tags modified, the event is forwarded to a central event bus where a Lambda evaluates and applies alarms.

The reconciliation Lambda still exists — it runs daily as a safety net — but it catches edge cases rather than doing the heavy lifting.

## What changed

- **Remediation time**: From up to 30 minutes to under 60 seconds
- **Lambda invocations**: Dropped significantly — we only run when something actually happens
- **Code complexity**: The event-driven Lambda handles one instance at a time, not a full cross-account sweep
- **Terraform**: The module became simpler because each component has a single, clear responsibility

## When cron still wins

Event-driven isn't always the answer. Use scheduled runs when:

- There's no reliable event source for the change you care about
- You need a full reconciliation sweep (drift detection, for example)
- The event volume would be higher than the polling cost

But for "react when something changes" — which is what most compliance automation is doing — EventBridge is the better tool.

## Getting started

If you're currently running a polling Lambda and want to shift:

1. Identify the AWS API action that triggers the change you care about
2. Create an EventBridge rule matching that event pattern
3. Keep your existing Lambda as a daily reconciliation fallback
4. Deploy the rule to member accounts via StackSets

The two patterns complement each other. Events handle the real-time path, scheduled runs handle the "trust but verify" path.
