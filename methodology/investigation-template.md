# Technical Investigation Template / Runbook

[Methodology](README.md) · [Evidence criteria](evidence-and-validation.md)

> Copy this template for a new case. The fields below are prompts for documenting an investigation.

## 1. Identification

| Field | Information |
|---|---|
| Identifier | INC-XXX / SEC-XXX / CHG-XXX |
| Case type | Real / Simulated / Planned |
| Status | Open / Investigating / Escalated / Resolved / Documented |
| Service | [Service or component] |
| Priority | [Impact and urgency, subject to the applicable SLA] |
| Environment and scope | [Non-sensitive information only] |

## 2. Symptom and business impact

**Reported symptom** — [What the user sees, without assuming cause].

**Impact** — [Unavailable service, affected people or processes].

**Expected result** — [How we will recognize recovery].

## 3. Evidence and diagnostics

| Check | Hypothesis tested | Observed result | Status |
|---|---|---|---|
| [Query] | [Assumption] | [Observed result or pending execution] | Verified / Untested |

Record exact commands for completed checks and label proposed commands. Sanitize account names, IP addresses, tokens, and keys before publishing output.

## 4. Hypotheses

**Primary** — [Hypothesis and supporting evidence].

**Alternatives** — [Other plausible causes].

**Root cause** — Confirmed / Probable / Undetermined.

Keep plausible explanations labeled as hypotheses until the evidence confirms them.

## 5. Action, risk, and authorization

**Proposed action** — [Smallest change].

**Operational and security risk** — [Availability, privacy, access, side effects].

**Authorization** — [Pending / Approved / Not required for this read-only check].

**Rollback or recovery** — [Applicable steps and prerequisites].

## 6. Outcome validation

- [ ] The affected function works again.
- [ ] The completed checks found no relevant side effects.
- [ ] The user or service owner confirmed the result where applicable.
- [ ] The evidence captures the before and after states.
- [ ] Limitations and residual risks are recorded.

## 7. Communication and closure

**User update** — [Brief, clear, and factual].

**Technical closure** — [What was verified, what is outstanding, and whether escalation occurred].

**Lessons learned** — [Preventive actions or runbook improvements].
