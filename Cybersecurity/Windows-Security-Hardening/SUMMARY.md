# Windows Security Assessment — Technical Summary

[Portfolio](../../README.md) · [Project and evidence](README.md) · [Detailed assessment](docs/security-assessment.md) · [Validation plan](docs/validation-plan.md) · [Methodology](../../methodology/README.md)

**Type** — Windows workstation assessment. Most checks were read-only and the owner approved one specific change.

**Status** — Assessment and documentation completed across 30 steps. Further hardening and follow-up checks are pending.

## Objective and method

I assessed native Windows controls across Defender, firewall, auditing, PowerShell, BitLocker, TPM, and Secure Boot. I documented the baseline, interpreted evidence, assessed risk, confirmed the authorized change, and verified its outcome.

## Key findings

| Area | Evidence-backed result | Limitation |
|---|---|---|
| Windows Firewall | Profiles enabled. ActiveStore showed inbound Block and outbound Allow. | Specific rules and live traffic were outside the test. |
| Microsoft Defender | Antivirus and real-time protection active. Selected advanced-protection indicators verified. | Effectiveness against real threats was outside the test. |
| Process auditing | Process Creation reported **No Auditing**. | Visibility gap documented. Policy stayed unchanged. |
| Security log | CHG-001 increased the limit from **20 to 128 MiB** and the configuration was verified. | The immediate span was **1.96 days**. The **seven-day** goal is pending. |
| BitLocker C: | Fully decrypted and protection Off. | Encryption requires backup and recovery preparation. |
| TPM and boot | TPM ready, UEFI and GPT verified, and Secure Boot False. | Firmware was assessed through read-only checks. |
| PowerShell | Operational channel active, **5.05 days** of observed history, and 4104 events present. | Complete logging coverage remains undetermined. |
| Module Logging | The queried HKLM/HKCU policy keys were absent and the 4103 query returned zero. | Effective coverage remains undetermined. |
| Transcription | HKLM/HKCU policy keys absent at the queried paths. | Transcription files were outside the review. |

## Authentication and privilege events

Logon auditing was configured for Success and Failure. A 24-hour query returned **235 event 4624 records** and no event 4625 group. The 4624 records included **229** type **5 (Service)**, **4** type **11 (CachedInteractive)**, and **2** type **7 (Unlock)**.

A separate query returned **233 event 4672 records** for special privileges assigned to new logons. The figures count records. Attribution to unique people or incidents requires additional evidence.

## Implemented change — CHG-001

The approved scope covered only the Security log maximum, from **20 MiB to 128 MiB**. The new value was verified. Seven days of retained history still require a follow-up measurement. BitLocker, Secure Boot, and process creation auditing remained outside the implementation.

## Priorities

1. Prepare verified backups, recovery procedures, and key custody before evaluating encryption or firmware changes.
2. Measure actual Security log retention again after enough time has elapsed.
3. Evaluate a process creation auditing pilot with approval, a privacy and performance review, and rollback criteria.
4. Define PowerShell logging requirements and check actual coverage.
5. Review aggregate authentication trends and scope deeper investigation to authorized cases.

## Closure

The baseline assessment is complete. Certification and full endpoint hardening were outside its scope. The review did not confirm an incident. The detailed documents distinguish observed evidence, authorized changes, and pending controls.
