# PST File Health Checklist

> A step-by-step reference for verifying the integrity of a Microsoft Outlook PST file **without opening Outlook**.
> This is useful when Outlook is unavailable, the PST is from a legacy system, or you suspect corruption before migration.

**Author:** Leena Taylor Paul | **Last updated:** 2025-01 | **License:** CC0

---

## Why Check PST Health?

A PST (Personal Storage Table) file can become corrupted due to:
- Abrupt Outlook/system shutdowns while the file was open
- Exceeding the 2 GB size limit (Outlook 2002 and earlier) or 50 GB (Outlook 2003+)
- File system errors, bad sectors, or incomplete copy/move operations
- Network drive disconnections during sync
- Virus scanner interference during write operations

Corrputed PSTs can cause permanent data loss if migrated or opened without first validating integrity.

---

## Pre-Check: Gather Basic File Information

- [ ] Record the full file path
- [ ] Note the file size (flag if > 4 GB as Outlook 2003-2016 max is ~50 GB; Outlook 2002 max is 2 GB)
- [ ] Note the file extension: `.pst` = personal store, `.ost` = offline store (different repair tools needed)
- [ ] Confirm the file system: NTFS recommended; FAT32 has a 4 GB per-file limit which can silently truncate large PSTs
- [ ] Check for a `.pst.tmp` or `.pst.log` companion file — this indicates Outlook was open when the system shut down unexpectedly

---

## Step 1: File-Level Integrity Check

### 1.1 Verify file is not zero-bytes or truncated
- [ ] Open File Explorer → right-click → Properties → confirm size > 0 bytes
- [ ] Compare the size to the expected backup size (if known)

### 1.2 Check for file lock
- [ ] Ensure Outlook is fully closed before proceeding
- [ ] On Windows: open Task Manager → check for `OUTLOOK.EXE` or `lync.exe` processes still running
- [ ] Try renaming the PST file temporarily — if rename fails, a process still has the file locked

### 1.3 Compute a checksum (optional but recommended)
- [ ] Run `certutil -hashfile yourfile.pst SHA256` in Command Prompt
- [ ] Record the hash; compare before/after any repair or migration step

---

## Step 2: Run Microsoft's Inbox Repair Tool (ScanPST.exe)

ScanPST is Microsoft's first-party PST diagnostic and repair utility. It does **not** require Outlook to be installed and running — only the Office files need to be present.

### Locate ScanPST.exe

Default locations by Office version:

| Office Version | Typical Path |
|---|---|
| Office 365 / 2019 (32-bit) | `C:\Program Files (x86)\Microsoft Office\root\Office16\` |
| Office 365 / 2019 (64-bit) | `C:\Program Files\Microsoft Office\root\Office16\` |
| Office 2016 | `C:\Program Files (x86)\Microsoft Office\Office16\` |
| Office 2013 | `C:\Program Files (x86)\Microsoft Office\Office15\` |
| Office 2010 | `C:\Program Files (x86)\Microsoft Office\Office14\` |
| Office 2007 | `C:\Program Files (x86)\Microsoft Office\Office12\` |

### Run the Scan
- [ ] Double-click `SCANPST.EXE`
- [ ] Click **Browse** and select the PST file
- [ ] Click **Start** — the scan runs in multiple phases (typically 8 phases)
- [ ] If errors are found, ScanPST will report them; **do not repair yet** — read results first
- [ ] Check the checkbox **"Make a backup of scanned file before repairing"** before clicking Repair
- [ ] Click **Repair** if errors were found
- [ ] Run ScanPST a **second time** after the first repair — complex corruption sometimes requires multiple passes

### ScanPST Limitations
- ScanPST fixes structural header corruption but **cannot recover deleted items**
- ScanPST creates a `.bak` backup of the original before repair (same folder, same name, `.bak` extension)
- If ScanPST itself crashes or reports it cannot repair the file, the PST has severe corruption — consider a specialist tool

---

## Step 3: Post-Repair Validation

- [ ] Open the repaired PST in Outlook and navigate to the **Deleted Items** or **Lost and Found** folder — ScanPST moves recovered orphaned items here
- [ ] Check that folder counts match expectations (Inbox count, Sent count, etc.)
- [ ] Search for a known email subject that should be present — confirm it is retrievable
- [ ] Verify attachments open correctly on a sample of emails
- [ ] Confirm calendar items are intact

---

## Step 4: Size and Format Checks

- [ ] If the PST is **ANSI format** (created by Outlook 2002 or earlier): max safe size is ~1.8 GB; migrate to Unicode format before further use
- [ ] To check format: open Outlook → File → Account Settings → Data Files → Properties → format shown as "Personal Folders" (ANSI) or "Office Outlook Personal Folders" (Unicode)
- [ ] If ANSI: export to a new Unicode PST using Outlook's Import/Export wizard before migrating

---

## Step 5: Document Your Results

For forensic, e-discovery, or enterprise migration work, document the following:

| Item | Value |
|---|---|
| PST file name | |
| File size (bytes) | |
| SHA-256 hash (pre-repair) | |
| SHA-256 hash (post-repair) | |
| ScanPST result | Pass / Errors found / Could not repair |
| Number of repair passes run | |
| Backup .bak file location | |
| Date/time of check | |
| Performed by | |

---

## Common Error Messages and What They Mean

| Error / Symptom | Likely Cause | Action |
|---|---|---|
| "Cannot open the Outlook Window" | PST is corrupt or locked | Run ScanPST |
| "The file [name].pst is not a personal folders file" | File is OST, not PST — or header is corrupt | Verify extension; run ScanPST on OST separately |
| "Errors have been found" in ScanPST | Header or B-tree corruption | Proceed with repair; run twice |
| ScanPST crashes mid-scan | Severe corruption | Try on a copy; consider third-party tool |
| Items appear missing after repair | Items were in corrupt blocks | Check Lost and Found folder in Outlook |
| PST opens but some folders show 0 items | Folder descriptor table corrupt | Third-party recovery tool may recover items |

---

## Related Notes in This Repository

- [Email File Format Reference](./email-file-formats.md) — Differences between PST, OST, MBOX, EML, OLM
- [Email Forensics Methodology](./forensics-methodology.md) — Header analysis, IP tracing, chain-of-custody

---

*CC0 — No rights reserved. Use freely.*
