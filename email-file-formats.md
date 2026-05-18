# Email File Format Reference

A practical reference for the five email container formats most commonly encountered in mailbox migration, archival, and forensic work: PST, OST, MBOX, EML, and OLM.

Author: Leena Taylor Paul | License: CC0

## Quick comparison

| Format | Vendor | Stores | Single/Multi message | Typical extension | Encoding |
|--------|--------|--------|----------------------|-------------------|----------|
| PST | Microsoft | Mail, calendar, contacts, tasks, notes | Multi | .pst | ANSI (legacy) or Unicode |
| OST | Microsoft | Cached server copy of an Exchange/M365 mailbox | Multi | .ost | Unicode |
| MBOX | Unix / many | Mail only | Multi (concatenated) | .mbox, .mbx | 7-bit / 8-bit text |
| EML | RFC 5322 / many | A single message | Single | .eml | MIME |
| OLM | Microsoft (Mac) | Mail, calendar, contacts | Multi | .olm | Proprietary, similar to a ZIP |

## PST (Personal Storage Table)

Microsoft Outlook's local archive format. Internally a B-tree of node and block tables. Two variants:

- ANSI (Outlook 97–2002): 2 GB hard cap, often silently corrupts near the limit.
- Unicode (Outlook 2003+): default 20 GB, raised to 50 GB in Outlook 2010+.

Notable properties:

- Can be password-protected, but the password is trivially recoverable because the file content itself is not encrypted.
- Stores deleted-items soft-deletes until compaction.
- Tools: `scanpst.exe`, `libpff` / `pypff`, `readpst`.

## OST (Offline Storage Table)

A local cached replica of a server mailbox (Exchange or Microsoft 365). Structurally similar to PST but tied to the originating profile and account credentials. An OST is not portable: opening it on a different machine or profile requires conversion (to PST, MBOX, or PDF) because the file cannot be re-attached without the original mailbox identity.

Forensic notes:

- An OST may contain items that have since been deleted from the server.
- Synchronization gaps mean OST contents can diverge from the live mailbox.
- The companion `.ost.tmp` or `OutlookSyncIssues` artifacts can be useful.

## MBOX

A flat-file format where messages are concatenated and separated by a `From ` line at the start of each message. Several dialects exist:

- mboxo (original): no length field, `From ` lines in bodies are escaped with `>`.
- mboxrd: same idea but uses `>From ` recursively for safety.
- mboxcl / mboxcl2: include a `Content-Length` header.

Used by Thunderbird (one MBOX per folder), Apple Mail (older versions), Eudora, and most Unix mail tools. Large MBOX files (>4 GB) are common and benefit from splitting by year or by message count for processing.

## EML

A single RFC 5322 / MIME message saved to disk. The file is plain text with headers, a blank line, and a body that may include MIME parts and base64-encoded attachments. EML is the most portable forensic exchange unit: every mail client, library, and analysis tool can read it.

## OLM

The export format of Outlook for Mac. Internally a ZIP archive containing XML descriptions of mail items, calendar entries, and contacts, with attachments as separate files. Not directly readable by Outlook for Windows; conversion to PST or MBOX is required for cross-platform migration.

## Choosing a working format for forensic processing

| Goal | Recommended working format |
|------|----------------------------|
| Per-message hashing and review | EML (one file per message) |
| Bulk text indexing and grep | MBOX |
| Re-import into Outlook | PST (Unicode) |
| Long-term archival, vendor-neutral | EML in a structured folder tree, plus a manifest |

## Related notes

- [PST Health Checklist](pst-health-checklist.md)
- [Email Forensics Methodology](forensics-methodology.md)
