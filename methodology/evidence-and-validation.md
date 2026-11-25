# Evidence Quality, Validation, and Change Control

[Methodology](README.md) · [Template](investigation-template.md)

## Classifying claims

| Classification | When I use it |
|---|---|
| **Verified** | A documented query or functional test confirmed the result. |
| **Observed** | A record or value was captured. Its cause remains a separate question. |
| **Inferred / Hypothesis** | A possible explanation remains unconfirmed. |
| **Proposed** | A procedure or improvement is documented for future execution. |
| **Simulated** | An explicitly fictional scenario. |
| **Pending** | A result, validation, or approval is still outstanding. |

## Quality checks

1. Preserve observed values, timestamps, error codes, and original console output.
2. Distinguish control configuration, effectiveness, and actual coverage.
3. Pair each command marked as executed with the result it produced.
4. Confirm root cause only when the evidence supports it.
5. Treat event counts as records. Assess distinct users, incidents, and malicious activity separately.
6. Document authorization, scope, backup, and rollback when evaluating a change.
7. Check links, diagrams, and references, and redact private information.

## Example: implementing and validating a change

In the [Windows Security Hardening lab](../Cybersecurity/Windows-Security-Hardening/README.md), **CHG-001** increased the maximum Security event log size from **20 MiB to 128 MiB**. The new capacity was verified. The seven-day retention target still needs a time-separated measurement.

PowerShell 4104 and logon 4624 events document recorded activity. Coverage and incident attribution require additional evidence.

## Protecting the documentation

Keep passwords, recovery keys, tokens, private raw logs, real user lists, and sensitive paths out of the public portfolio. Procedures can preserve the original command syntax. Sanitize screenshots and observed output before publication.
