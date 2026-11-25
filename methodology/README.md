# How I Work — IT Support and Investigation Methodology

[Portfolio](../README.md) · [Investigation template](investigation-template.md) · [Evidence criteria](evidence-and-validation.md)

## Purpose

I use this method to document IT support issues, security investigations, and systems administration decisions. It draws on incident and change management practices. It is a personal workflow for the portfolio.

## Workflow

1. **Intake and scope:** Identify symptoms, impact, urgency, affected services, boundaries, and authorizations.
2. **Evidence collection:** Start with the smallest useful checks, preferably read-only. Record the source, date, and context where available.
3. **Hypotheses and diagnosis:** Separate observations from possible explanations, compare alternatives, and start with the lowest-risk checks.
4. **Decision and change control:** Consider service continuity, privacy, permissions, backup, and rollback before changing a system.
5. **Resolution and validation:** Apply authorized actions and verify the expected outcome and possible side effects.
6. **Documentation and follow-up:** Record what worked, what remains unverified, when to escalate, and what to change for future incidents.

## Three ways I apply the method

| Type | What it means | Portfolio example |
|---|---|---|
| Resolved real case | There is evidence of execution and functional recovery. The root cause may remain open. | [INC-001 — Windows updates](../IT-Support/Windows-Troubleshooting-Lab/README.md) |
| Documented simulation | A proposed investigation with an explicitly fictional scenario. | [INC-002 — Microsoft 365 / MFA](../IT-Support/Service-Desk-Incident-Management/incidents/INC-002-M365-Authentication.md) |
| Technical assessment with controlled changes | I establish a baseline, prioritize risks, and track implemented and pending actions. | [Windows Security Hardening](../Cybersecurity/Windows-Security-Hardening/README.md) |

## Closure criteria

- **Resolved:** Functional checks confirm recovery. The evidence may support several possible causes.
- **Escalated:** The investigation needs permissions, teams, or decisions outside its scope.
- **Assessment completed:** Findings and recommendations are documented. Proposed changes may remain pending.
- **Simulation documented:** The diagnostic procedure and expected evidence are recorded as part of a fictional case.

## Working principles

**Evidence before assumptions · Least privilege · Privacy by design · Authorized changes · Independent validation · Clear communication.**

Public cases exclude customer data, credentials, private logs, and sensitive identifiers. Each case identifies whether it is real, simulated, or planned.
