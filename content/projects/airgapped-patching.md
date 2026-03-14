---
title: "Air-Gapped VPC Patch Management"
date: 2024-12-01
tag: "Security"
tech: ["SSM Patch Manager", "WSUS", "AMI Pipelines", "VPC"]
summary: "Solved patching for isolated VPC instances with no internet access by evaluating internal repo mirrors, WSUS on EC2, and pre-patched AMI replacement strategies."
---

## The Problem

A number of EC2 instances ran in fully isolated VPCs with no internet access — no NAT gateway, no internet gateway, no proxy. These instances still needed regular OS patching for compliance, but SSM Patch Manager's default behaviour requires access to public update repositories.

## Options Evaluated

### 1. Internal Repository Mirror (Linux)

For Linux instances, the most practical option was hosting an internal mirror of the required package repositories (e.g. Amazon Linux extras, RHEL repos) on an S3 bucket or an internal EC2 instance accessible via VPC endpoints.

**Pros**: Native package manager integration, granular control over available packages, works with SSM Patch Manager via custom baselines.

**Cons**: Requires ongoing sync to keep the mirror current, storage costs for full repo mirrors.

### 2. WSUS on EC2 (Windows)

For Windows instances, running Windows Server Update Services (WSUS) on an EC2 instance within the VPC (or accessible via peering/Transit Gateway) provides a fully internal update source.

**Pros**: Native Windows Update integration, granular approval workflows, well-understood by Windows admins.

**Cons**: WSUS itself needs internet access (or a sync chain) to pull updates from Microsoft, requires its own maintenance and monitoring.

### 3. Pre-Patched AMI Replacement

Rather than patching in place, build a pipeline that produces patched AMIs on a regular cadence. Instances are replaced rather than updated — an immutable infrastructure approach.

**Pros**: Eliminates patch drift entirely, clean instances every cycle, works regardless of network topology.

**Cons**: Requires stateless application design or external state management, longer rollout time, more complex pipeline.

## Recommendation

The realistic solution was a hybrid: **internal repo mirrors for Linux** (synced via a scheduled Lambda in an internet-connected VPC, pushed to S3, accessed via VPC endpoint) and **WSUS on EC2 for Windows** with a sync chain from an internet-connected instance.

For workloads that could support it, the **pre-patched AMI pipeline** was recommended as a longer-term target, particularly for auto-scaled workloads where instance replacement is already the norm.

## Key Takeaway

There's no single answer for air-gapped patching — it depends on the OS, the application architecture, and how much operational overhead the team can absorb. The important thing is to make the decision deliberately rather than letting isolated instances quietly fall out of compliance.
