---
title: "Lessons from Migrating 9TB of File Shares to FSx"
date: 2026-03-10
summary: "The discovery process, archival strategy, and gotchas nobody warns you about when moving Windows file shares off EC2."
tags: ["aws", "fsx", "migration", "powershell"]
---

Migrating a Windows file server sounds straightforward until you're staring at 9TB of data across 14 shares and trying to work out what's actually worth moving.

This is what I learned doing exactly that — moving a legacy EC2-hosted Windows file server to FSx for Windows File Server, with a detour through S3 Glacier for the data nobody was using.

## Start with discovery, not migration

The temptation is to spin up FSx, robocopy everything across, and call it done. Resist that. You'll end up paying FSx prices for terabytes of data that hasn't been touched in years.

I wrote a PowerShell script to scan every share and classify files by age. This immediately surfaced that a significant portion of the data was cold — files that hadn't been written to in over two years.

## The LastAccessTime trap

Here's the gotcha that cost me a day: the server had `DisableLastAccess` set to `1`. This is a common Windows performance optimisation, but it means `LastAccessTime` is unreliable — it wasn't being updated when files were read.

That left `LastWriteTime` as the only trustworthy timestamp. It's a reasonable proxy (if nobody's modified a file in two years, it's probably cold), but it's not perfect. A file that's read daily but never edited would appear cold.

**The fix**: I enabled `LastAccessTime` tracking early in the project timeline and let it run for a few weeks before the final classification scan. This gave us a more accurate picture before committing to the archival decisions.

**Lesson**: Check `fsutil behavior query DisableLastAccess` on day one of any file migration project.

## Archive before you migrate

With the data classified, the approach was:

1. Archive cold data to S3 Glacier (cheap, still retrievable if needed)
2. Migrate only active data to FSx
3. Keep the original EC2 instance read-only for a transition period

This significantly reduced the FSx storage footprint and brought the monthly cost down to something sensible.

## Things I'd do differently

- **Automate the archival pipeline end-to-end**: I used a semi-manual process with AWS DataSync. Next time I'd script the full workflow including verification and cleanup.
- **Set up monitoring on FSx from day one**: Storage growth on FSx can surprise you. CloudWatch alarms on free storage space are essential.
- **Communicate the archive process to users early**: People get nervous when they hear "we're archiving your files." Setting expectations about retrieval times and the safety net of Glacier avoids unnecessary panic.

## Was FSx worth it?

Yes. Automated backups, native AD integration, no more patching a Windows Server instance, and the storage scales without us managing disks. The migration was a few weeks of focused work, but the operational overhead dropped permanently.
