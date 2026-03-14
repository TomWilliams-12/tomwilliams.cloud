---
title: "EC2 to FSx File Share Migration"
date: 2025-01-15
tag: "Migration"
tech: ["FSx", "S3 Glacier", "PowerShell", "EC2", "Storage Gateway"]
summary: "Led the migration of a 9TB Windows file share (14 shares) from EC2 to FSx for Windows, including a cold-data archival strategy to optimise storage costs."
---

## The Problem

The organisation was running a Windows file server on EC2 hosting approximately 9TB of data across 14 shares. This was expensive, operationally brittle (single instance, manual backups), and didn't scale. The goal was to move to FSx for Windows File Server for native SMB support, automated backups, and better integration with Active Directory.

However, migrating 9TB directly to FSx would mean paying for storage that included years of cold data nobody was actively using.

## Discovery Phase

I built a PowerShell discovery script (`Scan-FileShareVolumes.ps1`) to analyse all 14 shares and classify data by age. A complication: the server had `DisableLastAccess` enabled, meaning we couldn't rely on `LastAccessTime` to determine which files were genuinely in use. Classification had to rely on `LastWriteTime` instead.

### Key findings from the scan:

<!-- TODO: Add your actual numbers here -->
- Total data: ~9TB across 14 shares
- Estimated cold data (not written to in 12+ months): X TB
- Estimated archival candidates (not written to in 24+ months): X TB

## Archival Strategy

Before migrating to FSx, cold data was identified for archival to S3 Glacier, significantly reducing the FSx storage footprint and ongoing costs.

The approach:

1. **Enable LastAccessTime tracking** ahead of the migration window to get a more accurate picture of genuinely accessed files
2. **Run a second scan** after the tracking period to refine classification
3. **Archive cold data** to S3 Glacier using AWS DataSync or a scripted approach
4. **Migrate remaining active data** to FSx for Windows

## Outcome

<!-- TODO: Add actual cost savings and timeline -->

The archival-first approach reduced the data migrated to FSx, resulting in lower monthly FSx costs and a cleaner, more manageable file share environment.

## Lessons Learned

- **Always check `DisableLastAccess` early**: It fundamentally changes your data classification strategy
- **Don't trust file counts alone**: A small number of large files can dominate storage, while thousands of small files dominate migration time
- **Build the discovery tooling first**: The scan script paid for itself many times over in planning accuracy
