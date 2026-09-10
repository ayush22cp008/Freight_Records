# Chat46 / Day19 / Node7 / Phase1b
# Reviewer Observation 2 — Recovery Historical Decision Preservation Governance Decision

**Project:** Freight — AI Builders Hackathon  
**Node:** Node 7 — AI + Final Integration + Demo  
**Phase:** Phase 1b — Full 3-Portal UI/UX Redesign  
**Day:** Day 19  
**Chat:** Chat46  
**Date:** 2026-09-10  
**Decision owner:** ChatGPT — Architecture / Governance Brain  
**Approval authority:** Ayush  
**Status:** **APPROVED — NARROW PERSISTENT VERIFICATION HISTORY CAPABILITY AUTHORIZED**

---

## 1. Purpose

This decision authorizes the minimum architectural capability required to satisfy both approved requirements simultaneously:

```text
1. A rejected applicant must be able to recover and return to PENDING.
2. The previous completed Reviewer rejection must remain readable in Verification History.
```

The existing `freight_identities` model stores one current identity row per applicant and the current Reviewer History derives completed history from that current-state row. The implementation investigation verified that updating the row from `REJECTED` to `PENDING` necessarily removes that row from the current History query.

Therefore, the existing model cannot represent both the new current `PENDING` state and the previous completed `REJECTED` decision without additional persistent historical decision storage.

This is an explicit, narrow architecture exception.

---

## 2. Decision

**Ayush approves a minimal persistent verification-history capability.**

The project may introduce the smallest durable representation required to preserve completed Reviewer decisions independently from the applicant's current `freight_identities` state.

The preferred form is a dedicated verification-history record/table or the smallest equivalent normalized structure, subject to implementation evidence.

This approval does not authorize a general audit framework or generalized evidence-history redesign.

---

## 3. Required Semantics

The resulting system must distinguish:

```text
CURRENT APPLICATION STATE
freight_identities.verification_status

from

COMPLETED REVIEW DECISION HISTORY
historical Reviewer decision record(s)
```

After recovery:

```text
Historical record:
    REJECTED
    + rejection reason
    + review timestamp
    + associated evidence reference where supported

Current application:
    PENDING
    + new/current evidence
```

The prior rejection must not be silently converted into `PENDING` history or disappear from Verification History.

---

## 4. Minimal Authorized History Capability

Implementation may add only the fields/structure necessary to persist completed Reviewer decisions, including as applicable:

- applicant/identity reference;
- decision status (`VERIFIED` or `REJECTED`);
- rejection reason when rejected;
- decision timestamp (`reviewed_at` or justified equivalent);
- reference to the evidence associated with that completed decision, when the existing evidence model can safely provide it;
- minimal creation/update metadata required for deterministic ordering and integrity.

The implementation should avoid duplicating unrelated applicant profile data.

Do not introduce a full event-sourcing system, general audit log, or broad historical snapshot framework.

---

## 5. Recovery Semantics

The approved recovery lifecycle is:

```text
REJECTED
   ↓
Applicant chooses Re-upload Evidence
   ↓
New evidence submission
   ↓
Create/preserve completed rejection history
   ↓
Current identity → PENDING
   ↓
Reviewer Queue
   ↓
New Reviewer decision
```

The prior rejection is a completed historical decision and the new submission is a new current reviewable state.

A subsequent decision may create another completed history record rather than mutating the previous one.

No `UNDER_REVIEW` state is introduced.

---

## 6. Evidence Preservation

The previous rejection's evidence must remain resolvable from History where the locked Reviewer Blueprint requires access to submitted evidence from completed records.

Do not rely on the current `onboarding_evidence` row alone if recovery replaces that row.

The implementation must preserve a stable reference or otherwise provide a deterministic mechanism for the completed historical record to resolve its associated evidence.

If this requires a broader evidence-schema redesign than the narrow minimum described above, STOP and return to governance.

---

## 7. Recovery Submission Integrity

The recovery implementation must also correct the previously verified RLS partial-write problem.

A successful recovery submission must not leave:

```text
new evidence inserted
+ identity still REJECTED
```

The system must end in a coherent state:

```text
current identity = PENDING
current submission/evidence = new corrected evidence
historical rejection = preserved independently
```

The exact authorization and write mechanism must remain minimal and server-side where privileged access is required.

---

## 8. Reviewer History Requirements

The locked Reviewer History surface must remain capable of:

- listing completed Verified and Rejected decisions;
- newest-first ordering by decision timestamp;
- deterministic selected-record detail;
- displaying rejection reason when present;
- opening associated evidence for the selected completed decision where supported;
- read-only behavior for completed records.

Recovery must not create duplicate ambiguous history records for the same decision.

---

## 9. Security Boundary

Historical records must remain readable only through the existing authorized Reviewer path.

Applicants must not gain access to other applicants' completed verification records.

Applicant recovery must remain owner-scoped.

No broad administrative permission expansion is authorized.

Any new history read/write path must enforce the existing Reviewer authorization model or a minimal equivalent explicitly justified by source evidence.

---

## 10. Explicitly Not Authorized

This decision does **not** authorize:

- generalized audit logging;
- event sourcing;
- full evidence-history redesign;
- broad evidence versioning architecture;
- broad RLS redesign;
- generalized transaction framework;
- authentication changes;
- role-model changes;
- new persistent `UNDER_REVIEW` state;
- automated verification;
- applicant self-approval;
- Reviewer authority expansion;
- navigation redesign;
- visual redesign;
- evidence-load gating work;
- Driver changes;
- Company changes;
- trip/delivery changes;
- marketplace changes;
- AI behavior changes;
- C-05;
- R-03;
- unrelated API-contract changes.

Any requirement beyond this exact history-preservation capability is a stop condition.

---

## 11. Required Implementation Validation

The implementation must demonstrate all of the following:

### A. Rejection preservation

```text
Reject applicant with reason + evidence
→ completed History record persists
```

### B. Recovery

```text
Rejected applicant
→ Re-upload Evidence
→ submit corrected evidence
→ current identity becomes PENDING
```

### C. Queue re-entry

```text
PENDING applicant
→ appears in Reviewer Queue
```

### D. History preservation after recovery

```text
Old rejected record
→ remains in History
→ old reason visible
→ old decision timestamp visible
→ old evidence resolvable where required
```

### E. New decision separation

After a later Reviewer decision, the new completed decision must not overwrite the previous completed decision record.

### F. Failure consistency

Failed recovery submission must not falsely report success and must not leave an incoherent partial state without explicit evidence/handling.

### G. Security

Applicants can only operate on their own current recovery data; Reviewer-only history remains protected.

---

## 12. Implementation Stop Conditions

Antigravity must STOP if implementation requires:

- broad schema redesign;
- generalized history/audit architecture;
- broad RLS changes;
- a new lifecycle model beyond current state + completed history;
- destructive migration of existing verification history without a safe preservation plan;
- unrelated portal/system changes;
- any conflict with the locked Reviewer Blueprint.

Return the exact blocker for a new decision.

---

## 13. Final Approved Decision

> **Ayush approves a narrow persistent verification-history capability so the system can represent a completed Reviewer rejection independently from the applicant's current `PENDING` recovery state.** The implementation may introduce the minimum historical decision structure necessary to preserve prior status, reason, decision timestamp, and evidence linkage where required, while making recovery operational and returning the applicant to the existing Reviewer Queue. No general audit/history redesign is authorized.

**Ayush approval:** YES  
**Persistent history capability:** APPROVED — NARROW  
**Current state + completed history separation:** REQUIRED  
**REJECTED → PENDING:** AUTHORIZED  
**Prior rejection preservation:** REQUIRED  
**Broad history/audit redesign:** NOT AUTHORIZED  
**Observation 2:** OPEN — IMPLEMENTATION MAY PROCEED WITH NARROW ARCHITECTURE EXCEPTION

---

## 14. Next Action

Create a dedicated implementation handoff based solely on this governance decision. The handoff must require the minimum persistent verification-history capability plus the recovery submission correction and manual verification evidence.
