---
title: "Event-Driven CloudWatch Alarm Remediation"
date: 2025-02-01
tag: "Automation"
tech: ["Terraform", "Lambda", "EventBridge", "CloudWatch", "SNS", "StackSets"]
summary: "Designed a central Terraform module with event-driven Lambdas to enforce CloudWatch CPU alarms across a multi-account AWS organisation, resolving a compliance gap."
---

## The Problem

A compliance audit flagged that CloudWatch CPU alarms were not consistently deployed across EC2 instances in our multi-account AWS organisation. Instances were being launched across dozens of accounts and the existing manual process couldn't keep pace.

## The Approach

I designed a solution with three key components:

### 1. Central Terraform Module

A reusable `cloudwatch-cpu-alarms` module that could be deployed consistently across accounts. The module supports a **three-tier alarm threshold priority system**:

- **Instance-level**: EC2 tags allow teams to set custom thresholds per instance
- **Environment-level**: Default thresholds per environment (production gets tighter thresholds than dev)
- **Global fallback**: Organisation-wide defaults catch anything without explicit configuration

### 2. Event-Driven Detection

EventBridge rules deployed via StackSets to every member account. When a new EC2 instance launches (or an existing one has its tags modified), an event triggers a Lambda function that evaluates whether alarms are in place and creates or updates them as needed.

### 3. Reconciliation Lambda

A scheduled Lambda performs a full sweep across all accounts, catching any instances that might have been missed by the event-driven path — for example, instances launched during a deployment issue or accounts that were onboarded before the StackSets were deployed.

## Notification Pipeline

Alarm state changes flow through SNS with a Lambda subscriber that formats and delivers notifications to Microsoft Teams channels, giving operations teams real-time visibility.

## Outcome

<!-- TODO: Add specific metrics once you have them — e.g. "Reduced compliance gaps from X to zero" or "Covered 100% of instances within 48 hours of deployment" -->

The solution brought the organisation into full compliance and eliminated the need for manual alarm management. New instances are covered automatically within minutes of launch.

## Key Decisions

- **EventBridge over CloudTrail polling**: Lower latency, native integration, and no additional cost for the event matching
- **StackSets for cross-account deployment**: Consistent deployment with automatic coverage of new accounts via delegated admin
- **Three-tier priority system**: Gives teams flexibility without sacrificing baseline coverage
