# Assessment Register — Windows Security Baseline

[Project](../README.md) · [Baseline guide](baseline-guide.md) · [Validation plan](validation-plan.md)

**Phase** — Assessment completed across Steps 1–30.  
**Change control** — **CHG-001 was the only implemented change and its configuration was verified.** Seven-day retention and the remaining improvements are pending.

## Control assessment

| Domain | Objective | Verified evidence | Risk or limitation | Status |
|---|---|---|---|---|
| Firewall | Limit unsolicited traffic | All three profiles enabled. Initial action values were `NotConfigured`. Effective `ActiveStore` defaults were inbound `Block`, outbound `Allow`. | Rule exceptions and live traffic were outside the review. | **Selected settings verified** |
| Defender Antivirus | Protect the endpoint | Service, antivirus, real-time protection, Tamper Protection, and NIS active. Signatures 0 days old, updated 2026-11-17 at 09:41:14 console time. | Exclusions, signature behavior, and malware efficacy were outside the review. | **Indicators verified** |
| Advanced Defender settings | Check PUA, cloud protection, and early blocking | `MAPSReporting=2`, `SubmitSamplesConsent=1`, `PUAProtection=1`, `DisableBlockAtFirstSeen=False`. | Full cloud protection workflow was outside the review. | **Preferences verified** |
| Security log | Retain evidence for investigations | CHG-001 changed the maximum from **20 to 128 MiB**, `Circular`, with 33,989 records afterward. | Immediate span **1.96 days**, seven-day target pending. | **Configuration verified** |
| Process auditing | Record process creation | `Process Creation: No Auditing`. | May limit new 4688 events. | **Potential visibility gap; no approved change** |
| System-volume encryption | Protect data at rest | C: `FullyDecrypted`, `ProtectionStatus=Off`, `0%`. | Exposure on physical loss depends on context. | **Verified; encryption deferred** |
| TPM | Check hardware capability | `Present/Ready/Enabled/Activated=True`. | Backup and recovery-key readiness require separate checks. | **Verified** |
| Secure Boot | Check boot integrity configuration | `Confirm-SecureBootUEFI=False`, UEFI firmware. | Secure Boot disabled. | **Verified; change deferred** |
| System disk | Verify platform compatibility | Disk 0 GPT, disk-level `IsBoot=True` and `IsSystem=True`. | Firmware and individual partitions need separate review. | **Verified** |
| Script Block Logging | Review script block visibility | HKLM/HKCU keys absent at queried paths. 438 retained 4104 events, then 439 warnings in a later query. | Effective coverage and event origin undetermined. | **Metadata verified** |
| Module Logging | Review module telemetry | HKLM/HKCU keys absent. The 4103 query returned zero. | Non-terminating errors were suppressed and retention is circular. | **No 4103 retrieved; coverage unknown** |
| PowerShell Transcription | Review session transcription | HKLM/HKCU keys absent at queried paths. | Transcription files and other sources were outside the review. | **Policy paths checked** |
| PowerShell Operational | Retain PowerShell events | Enabled, 808 records across 11 event IDs, 15 MiB, Circular. | The **5.05-day** observed span does not establish a retention SLA. | **Channel and span verified** |
| Logon auditing | Record authentication | `Logon = Success and Failure`. | Actual coverage depends on retained events. | **Verified** |
| Authentication 4624/4625 | Review sign-in activity | 235 event 4624 records and no 4625 group in a 24-hour query. | Counts represent records. | **Aggregates verified** |
| 4624 logon types | Distinguish service and interactive sessions | Type 5: 229, type 11: 4, type 7: 2. | Service predominance alone cannot determine intent. | **Distribution verified** |
| Special privileges 4672 | Identify privilege assignments | 233 records in a 24-hour query. | The count does not establish misuse or distinct administrators. | **Count verified** |
| Local identity and access | Assess local role exposure | No measurement. | Requires a narrowly scoped, sanitized query. | **Not assessed** |
| Update posture | Identify missing updates | No measurement. | Requires a read-only check. | **Not assessed** |

## CHG-001 evidence

The only authorized modification was:

```powershell
wevtutil sl Security /ms:134217728
```

**Before:** `MaximumSizeInBytes=20971520` (20 MiB), `RecordCount=33959`, `Circular`.  
**After:** `MaximumSizeInBytes=134217728` (128 MiB), `IsEnabled=True`, `RecordCount=33989`, `Circular`.

The earliest-to-latest event span was **1.93 days before** and **1.96 days immediately after**. The queries were independent snapshots of a circular log. Event continuity, exactly 30 new events, and the seven-day retention goal require separate verification.

[Exact output and verification procedure](baseline-guide.md#chg-001--security-event-log-maximum-size-increase-implemented-and-configuration-verified).

## Finding priorities

| Planning priority | Finding | Assessment | Recommendation |
|---|---|---|---|
| High attention, recovery first | C: unencrypted, Secure Boot disabled, TPM ready, UEFI/GPT | Potential data and boot-chain exposure | Verify backup, recovery media, and key custody before authorizing separate changes |
| Medium, retention | 128 MiB Security limit verified, 1.96 days immediately observed | Seven-day coverage remains pending | Recheck endpoints after enough time and size based on event volume |
| Medium, detection | Process creation auditing disabled | Possible 4688 visibility gap | Evaluate a pilot with privacy, operational cost, and rollback criteria |
| Evaluate telemetry | PowerShell policy keys absent, 4104 events present | Effective coverage unknown | Review policy scope and use cases before adding logging |
| Monitor trends | 235 event 4624, 233 event 4672, 229 service logons | Counts are consistent with service activity and do not determine intent | Review aggregate patterns and scope any authorized correlation |
| Maintain | Active Defender and firewall indicators | Positive status for the inspected fields | Review configuration periodically |

These priorities sequence work. CVSS scoring and compliance certification were outside the assessment.

## PowerShell audit matrix

| Feature | HKLM | HKCU | Evidence |
|---|---|---|---|
| Script Block Logging | Key absent | Key absent | 438 event 4104 records initially, 439 warning records later |
| Module Logging | Key absent | Key absent | Zero event 4103 records returned |
| Transcription | Key absent | Key absent | Transcription files were outside the review |
| Operational | Outside these policy keys | Outside these policy keys | Enabled, 15 MiB, circular, 808 records, 5.05 days observed |

The queried policy paths cover only part of the configuration. The 4104 results establish recorded events. The 4103 query suppressed non-terminating errors.

## Interpreting authentication

Logon was configured to audit successes and failures. One 24-hour query returned **235 event 4624 records** and no 4625 group. A separate classification found **229 type 5 (Service, 97.4%), 4 type 11 (CachedInteractive), and 2 type 7 (Unlock)**.

Another query returned **233 event 4672 records**. Approximate volume agreement cannot establish one-to-one correspondence. A 4672 records privilege assignment to a session. Privilege use and malicious access require other evidence.

## Classification rules

- **Verified:** an executed check has an observed result.
- **Prior evidence:** an observation from the related Blue Team project.
- **Proposed:** a potential improvement awaiting approval and implementation.
- **Unknown:** a condition outside the observed evidence.
- **Not applicable:** requires a reason tied to the platform and environment.

## Assessment criteria

Evaluate each control against its purpose, device role, policy precedence, privacy, compatibility, and operational cost. An enterprise baseline difference requires contextual assessment.

Use bounded, sanitized records. This review excludes account data, tokens, IP addresses, customer information, and BitLocker secrets. The work assessed configuration and evidence. No incident was confirmed.

## References and related documents

- [Full 30-step guide and exact output](baseline-guide.md)
- [Validation plan and CHG-001](validation-plan.md)
- [Earlier event analysis](../../Windows-Security-Event-Analysis/README.md)
- [Windows security baselines — Microsoft Learn](https://learn.microsoft.com/en-us/windows/security/operating-system-security/device-management/windows-security-configuration-framework/windows-security-baselines)
- [auditpol /get](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/auditpol-get)

The evidence supports the checks documented here. Organizational compliance was outside the assessment.
