# Email Forensics Methodology

A practitioner-oriented walkthrough of how to examine an email message or mailbox for an investigation, e-discovery matter, or incident response. This is methodology, not legal advice.

Author: Leena Taylor Paul | License: CC0

## 1. Preserve before you analyze

The first rule is the same as in any digital forensics work: never operate on the original. Steps:

1. Acquire the source (PST, OST, MBOX, individual EMLs, server export).
2. Compute SHA-256 hashes of every source artifact.
3. Record acquisition metadata: who, when, from where, by what method.
4. Store the originals read-only; work on copies only.
5. Maintain an evidence log with one line per action taken.

## 2. Understand the headers

A modern email header contains far more than `From`, `To`, and `Subject`. The forensically interesting fields are:

| Header | What it tells you |
|--------|-------------------|
| Received (multiple) | The hop-by-hop path; read bottom to top |
| Return-Path | The envelope sender used for bounces |
| Message-ID | Unique identifier set by the originating server |
| DKIM-Signature | Cryptographic signature of selected headers and body |
| Authentication-Results | Receiving server's verdict on SPF, DKIM, DMARC |
| X-Originating-IP | Originating client IP (not always present) |
| X-Mailer / User-Agent | Sending client software |
| Date | Sender-claimed time; compare with Received timestamps |

## 3. Trace the path

Walk the `Received` chain from the last (top) to the first (bottom). Each hop should show `from` (claimed identity), `by` (receiving server), a protocol (`with ESMTPS`, etc.), and a timestamp. Key checks:

- Do the timestamps increase monotonically? Gaps or reversals indicate clock skew, header forgery, or transit through a backdated relay.
- Does the earliest `Received` show an IP in a residential block, a hosting provider, or a known relay? Resolve PTR and ASN.
- Are SPF/DKIM/DMARC results in `Authentication-Results` consistent with the claimed sender domain?

## 4. Verify the body

- Compare any displayed sender name against the actual `From` address.
- Inspect links: render the raw URL, not the anchor text. Watch for punycode, lookalike domains, and URL shorteners.
- Inspect attachments by hash, not by filename. Compare against threat intel sources where appropriate.
- For HTML bodies, check for tracking pixels and remote image loads.

## 5. Establish chain of custody

A minimum chain-of-custody record per artifact:

- Artifact identifier
- SHA-256 hash at each handoff
- Acquisition date/time (UTC) and time zone of acquiring system
- Method of acquisition (export, image, server-side eDiscovery hold)
- Custodian transfers (from / to / date / signature)
- Storage location and access controls

## 6. Document the analysis

Every conclusion should be reproducible. For each finding, record:

- The artifact and hash it came from
- The exact tool and version used
- The command or steps executed
- The output observed
- The interpretation, separated from the raw observation

## 7. Legal and compliance considerations (non-exhaustive)

- GDPR: even for forensic purposes, personal data should be minimized and access logged.
- Cross-border transfer of mailbox data may trigger additional controls.
- Some jurisdictions require explicit authorization to access an employee's mailbox even on company infrastructure.
- Treat anything that may end up in litigation as litigation-quality from the moment you touch it.

## Related notes

- [PST Health Checklist](pst-health-checklist.md)
- [Email File Format Reference](email-file-formats.md)
