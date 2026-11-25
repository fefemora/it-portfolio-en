# Change and Validation Plan — Windows Security Hardening

[Project](../README.md) · [Security assessment](security-assessment.md) · [Baseline guide](baseline-guide.md)

**Lab phase** — Steps 1–30 documented. Further changes require separate approvals.  
**Implemented action** — **CHG-001**, raising the maximum Security log size from 20 to 128 MiB.  
**Validation** — Configuration and immediate historical access verified. The seven-day target remains pending.  
**Outside implementation** — Process auditing, PowerShell logging, BitLocker, Secure Boot, Defender, and firewall changes.

## 1. Change-control method

1. Collect a **read-only baseline** and record collection times.
2. Define the security objective and reason for the change.
3. Verify ownership and domain, MDM, or local policy management.
4. Review user impact, remote access, privacy, and event volume.
5. Prepare recovery and rollback for the specific change.
6. Obtain authorization for a narrowly scoped modification.
7. Record the exact action and independently capture its after-state.
8. Verify functionality and document deviations or rollback.

This checklist describes future change control. The personal lab approval was limited to the observed action below.

## 2. CHG-001 — Security log capacity increase

### Problem and scope

The Security log had a **20 MiB** maximum in `Circular` mode. The immediate baseline covered about **1.93 days**. The proposed 128 MiB maximum would increase capacity for investigations.

The owner approved changing only the Security channel maximum. Process creation, PowerShell retention, firewall, Defender, C: encryption, and firmware settings were outside the approved scope.

### Verified before-state

**Executed query:**

```powershell
Get-WinEvent -ListLog Security |
    Select-Object LogName,
                  RecordCount,
                  MaximumSizeInBytes,
                  LogMode |
    Format-List
```

**Observed output:**

```text
LogName            : Security
RecordCount        : 33959
MaximumSizeInBytes : 20971520
LogMode            : Circular
```

The query omitted `IsEnabled`. Other checks had verified that the channel was enabled.

The earlier event-analysis snapshot held **33,898** records and covered about **2.03 days**. The CHG-001 baseline held **33,959** records and covered **1.93 days**, with **56.81 GiB** free on C:. The 61-record and 0.10-day differences (about 2 h 24 min) are comparisons between circular-log snapshots. They do not establish the exact number of new events or a particular rollover.

### Authorized execution

```powershell
wevtutil sl Security /ms:134217728
```

The command raised the limit from **20 MiB (20,971,520 bytes)** to **128 MiB (134,217,728 bytes)**.

### Post-change configuration check

```powershell
Get-WinEvent -ListLog Security |
    Select-Object LogName,
                  IsEnabled,
                  RecordCount,
                  MaximumSizeInBytes,
                  LogMode |
    Format-List
```

**Observed output:**

```text
LogName            : Security
IsEnabled          : True
RecordCount        : 33989
MaximumSizeInBytes : 134217728
LogMode            : Circular
```

The check confirmed **128 MiB**, enabled status, and Circular mode. The increase was **108 MiB** (6.4 times the original capacity). The counts of 33,959 and 33,989 came from independent snapshots. Exactly 30 new events and individual event continuity were not established.

### Pre-change retained interval

**Executed query:**

```powershell
$oldest = Get-WinEvent -LogName Security -Oldest -MaxEvents 1
$newest = Get-WinEvent -LogName Security -MaxEvents 1

[PSCustomObject]@{
    OldestEvent  = $oldest.TimeCreated
    NewestEvent  = $newest.TimeCreated
    RetainedDays = [math]::Round(
        ($newest.TimeCreated - $oldest.TimeCreated).TotalDays, 2
    )
}
```

**Observed output:**

```text
OldestEvent        NewestEvent         RetainedDays
-----------        -----------         ------------
16-11-2026 0:52:07 17-11-2026 23:06:29         1,93
```

### Immediate post-change check

```powershell
$oldest = Get-WinEvent -LogName Security -Oldest -MaxEvents 1
$newest = Get-WinEvent -LogName Security -MaxEvents 1

[PSCustomObject]@{
    OldestEvent = $oldest.TimeCreated
    NewestEvent = $newest.TimeCreated
    OldestRecordId = $oldest.RecordId
    NewestRecordId = $newest.RecordId
    RetainedDays = [math]::Round(($newest.TimeCreated - $oldest.TimeCreated).TotalDays, 2)
} | Format-List
```

**Observed output:**

```text
OldestEvent    : 16-11-2026 0:52:21
NewestEvent    : 18-11-2026 0:00:06
OldestRecordId : 2256643
NewestRecordId : 2291047
RetainedDays   : 1,96
```

Events predating CHG-001 remained accessible. The oldest observed timestamp moved 14 seconds, from 00:52:07 to 00:52:21 on November 16. The **1.96-day** span leaves the seven-day target unverified. The record IDs were captured as metadata and were not checked for gaps. The documented procedure includes no rollback or log clearing.

Assess possible loss of retained history before lowering the maximum back to 20 MiB. No rollback was run.

### CHG-001 status

| Field | Result |
|---|---|
| Approved scope | Security log maximum only, 20 → 128 MiB |
| Action | `wevtutil sl Security /ms:134217728` |
| After-state limit | **128 MiB**, verified |
| Channel enabled / mode | `True` / `Circular`, verified |
| Before interval | **1.93 days** observed |
| Immediate after interval | **1.96 days** observed |
| Seven-day target | **Pending verification** |
| Rollback | Not performed |
| Outcome | **Change and configuration verified. Long-term retention pending.** |

## 3. Proposed improvements awaiting implementation

| Candidate | Objective | Validation needed | Risks and prerequisites |
|---|---|---|---|
| Firewall | Review profile and rule protection | Effective state and appropriate connectivity checks | Remote service disruption |
| Defender | Review protection and signature health | Fresh status and preference output | External policies and product coexistence |
| Process auditing | Improve 4688 visibility | Before/after audit policy, new events, and volume | Privacy, sensitive commands, storage |
| PowerShell logging | Define Script Block, Module, and Transcription coverage | Effective state, events, access, and retention | Volume, confidentiality, and organizational policy |
| BitLocker | Protect C: at rest | Encryption and protection after-state, tested recovery | Recovery-key loss and boot issues |
| Secure Boot | Protect system startup | After-state and successful boot | Firmware, keys, Hyper-V, and recovery compatibility |
| Security retention | Reach seven days | Multiple time-separated measurements | Variable event volume and circular overwrite |

These candidates remained proposals.

## 4. Process creation auditing — Separate proposal

**Verified output:**

```text
Directiva de auditoría del sistema
Categoría o subcategoría                  Configuración
Seguimiento detallado
  Creación del proceso                    Sin auditoría
```

This setting can limit visibility of future **4688** events. The current query cannot establish the policy in effect when older 4688 records were created.

**Proposed pilot (not executed).** Evaluate Success auditing for process creation. Failure auditing and command-line capture are outside the initial scope. Check approval, AD/MDM policy, available storage, a timestamped before-state, privacy, and the observation window. More events could shorten the retained span.

**Proposed enablement:**

```powershell
auditpol /set /subcategory:"{0cce922b-69ae-11d9-bed3-505054503030}" /success:enable
```

**Proposed verification:**

```powershell
auditpol /get /subcategory:"{0cce922b-69ae-11d9-bed3-505054503030}"
```

After an authorized change, start an agreed benign process, check for new 4688 events with a bounded query, verify functionality, and monitor log growth. Validate configuration and event generation independently.

**Proposed rollback (not executed):**

```powershell
auditpol /set /subcategory:"{0cce922b-69ae-11d9-bed3-505054503030}" /success:disable
```

Use this rollback only if the saved baseline confirms Success was disabled and device policy permits the change. Recheck policy and preserve existing records. The pilot has no associated approved change.

## 5. BitLocker and Secure Boot — Deferred changes

### Verified baseline

| Component | Result |
|---|---|
| C: `VolumeStatus` | `FullyDecrypted` |
| C: `ProtectionStatus` | `Off` |
| C: `EncryptionPercentage` | `0` |
| TPM Present, Ready, Enabled, Activated | All `True` |
| Firmware | `Uefi` |
| System disk | Disk 0, `GPT`, disk-level `IsBoot=True`, `IsSystem=True` |
| Secure Boot | `False` |

Backup and recovery prerequisites remain unverified. Before an approved change:

1. Create and verify an independent volume backup.
2. Prepare and test recovery media.
3. Plan secure BitLocker recovery-key storage outside GitHub.
4. Verify Windows edition, firmware, signed boot, and Hyper-V compatibility.
5. Authorize firmware and encryption changes independently.
6. Verify boot, services, and recovery, and record deviations.

BitLocker, TPM, Secure Boot, partitions, and firmware were assessed without changes.

## 6. PowerShell telemetry — Pending recommendations

The Operational channel was enabled with **808 records**, a **15 MiB** circular maximum, and a **5.05-day** observed span. Queried HKLM/HKCU paths for `ScriptBlockLogging`, `ModuleLogging`, and `Transcription` had no policy keys. One check returned **438** event 4104 records. Another returned **439 warnings**. The 4103 query returned no records and suppressed non-terminating errors.

Effective coverage remains undetermined. Define detection objectives, policy precedence, privacy, and retention before approving additional logging. PowerShell settings stayed unchanged.

## 7. Authentication and privileges — Baseline observations

Logon auditing was configured for Success and Failure. A 24-hour query returned **235 event 4624 records**, with no 4625 matches, and the 4624 distribution was **229 type 5 (Service), 4 type 11 (CachedInteractive), and 2 type 7 (Unlock)**. An independent query returned **233 event 4672 records**.

These are event counts. They do not establish distinct people, incidents, or causal correlation. Authentication policies and permissions stayed unchanged.

## 8. Final state and follow-up

The baseline assessment and the configuration check for the one implemented change are complete. Full endpoint hardening and certification were outside the project scope.

- Measure Security retention again after enough time and compare it to seven days.
- Keep process auditing and PowerShell logging as separate proposals.
- Prepare backup and recovery before BitLocker or Secure Boot changes.
- Keep accounts, private identifiers, passwords, and recovery keys out of public evidence.
- Record each change's approval, execution, validation, and any rollback independently.

## Official references

- [Microsoft Learn — Security Compliance Toolkit](https://learn.microsoft.com/en-us/windows/security/operating-system-security/device-management/windows-security-configuration-framework/security-compliance-toolkit-10)
- [Microsoft Learn — Windows security baselines](https://learn.microsoft.com/en-us/windows/security/operating-system-security/device-management/windows-security-configuration-framework/windows-security-baselines)
- [Microsoft Learn — Audit Process Creation](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/audit-process-creation)

These guides support the recommendations. The implementation and verification claims come from the lab evidence.
