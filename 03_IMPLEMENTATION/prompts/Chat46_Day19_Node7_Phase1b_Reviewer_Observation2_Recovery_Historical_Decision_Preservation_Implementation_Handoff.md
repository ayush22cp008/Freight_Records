# Chat46 / Day19 / Node7 / Phase1b
# Reviewer Observation 2 — Recovery Historical Decision Preservation Implementation Handoff

**Project:** Freight — AI Builders Hackathon  
**Node:** Node 7 — AI + Final Integration + Demo  
**Phase:** Phase 1b — Full 3-Portal UI/UX Redesign  
**Day:** Day 19  
**Chat:** Chat46  
**Execution agent:** Antigravity  
**Status:** AUTHORIZED — NARROW IMPLEMENTATION

---

## 1. Governing Decision

Implement strictly from:

```text
00_PROJECT_CONTROL/DECISIONS/Chat46_Day19_Node7_Phase1b_Reviewer_Observation2_Recovery_Historical_Decision_Preservation_Governance_Decision.md
```

That governance decision authorizes a **narrow persistent verification-history capability** because the existing current-state model cannot simultaneously represent:

```text
current application = PENDING
previous completed decision = REJECTED
```

without losing the prior completed Reviewer decision from Verification History.

This handoff does **not** authorize a broader Reviewer, lifecycle, history, evidence, security, or platform redesign.

---

## 2. Problem To Correct

The currently verified recovery defect is:

```text
REJECTED applicant
→ re-upload evidence
→ new evidence may be inserted
→ identity update to PENDING is blocked by existing RLS path
→ API returns failure / partial-write risk
→ applicant does not coherently re-enter Reviewer Queue
```

The prior implementation investigation also established that current Reviewer History derives completed decisions from the current `freight_identities` row. Because there is one current identity row per applicant, changing that row from `REJECTED` to `PENDING` causes the prior rejection to disappear from the existing completed History model.

Therefore the implementation must separate:

```text
CURRENT APPLICATION STATE
freight_identities.verification_status

from

COMPLETED REVIEW DECISION HISTORY
persistent historical Reviewer decision record
```

---

## 3. Required Implementation Scope

### A. Inspect Before Changing

Before modifying anything, inspect the current source, migrations/schema, Reviewer History implementation, Reviewer review endpoint, onboarding recovery flow, evidence storage/query path, and relevant RLS/authorization rules.

Do not assume the investigation report is the final implementation shape. Confirm the current code and schema first.

### B. Add the Minimum Persistent History Capability

Introduce the smallest durable history structure required to persist completed Reviewer decisions independently from current applicant state.

Preferred form:

```text
dedicated verification-history record/table
```

or the smallest equivalent normalized structure supported by the existing schema.

The persistent completed-decision record must preserve, as applicable and without unrelated duplication:

- applicant/identity reference;
- completed decision status (`VERIFIED` or `REJECTED`);
- rejection reason when rejected;
- decision timestamp (`reviewed_at` or justified equivalent);
- stable evidence reference for the evidence associated with that completed decision, where the existing evidence model can safely support it;
- only the minimal metadata necessary for deterministic ordering and integrity.

Do not copy unrelated applicant profile data into history.

### C. Preserve Completed Rejection Before Recovery

When a Reviewer rejects an applicant, the completed rejection must be durably represented before recovery can replace the applicant's current state/evidence.

Required result:

```text
Historical record:
    REJECTED
    + prior rejection reason
    + prior decision timestamp
    + resolvable prior evidence reference where required

Current applicant state:
    REJECTED
```

The existing Reviewer decision behavior must remain valid for normal rejection/approval.

### D. Correct Recovery Submission

For a rejected applicant using the existing recovery action:

```text
REJECTED
→ Re-upload Evidence
→ submit corrected evidence
→ current identity = PENDING
→ Reviewer Queue eligibility restored
```

Correct the previously verified RLS-induced partial-write behavior.

A successful recovery submission must not produce:

```text
new evidence inserted
+ identity still REJECTED
```

The write/authorization path may be adjusted narrowly and server-side where privileged access is required.

Keep owner scoping intact.

### E. Evidence Integrity

The previous completed rejection must remain resolvable from Reviewer History after recovery.

The new/current evidence must be the evidence associated with the new current `PENDING` review.

Do not allow recovery to create duplicate or ambiguous evidence state for the same logical submission without a deterministic relationship.

Do not redesign the entire evidence model.

### F. Reviewer History Behavior

Reviewer History must continue to support the locked Blueprint semantics:

- completed Verified and Rejected decisions are listed;
- newest-first ordering by decision timestamp;
- deterministic selected-record detail;
- rejection reason shown when present;
- associated completed evidence can be opened where required;
- completed records are read-only.

After recovery, the prior rejected record must still appear in History even though the applicant's current state is `PENDING`.

After a later Reviewer decision, the new completed decision must become a separate historical record and must not overwrite the prior rejection record.

### G. Applicant Feedback

The recovery form/action must provide accurate submission outcome feedback.

On success, the applicant should receive a clear indication that the corrected evidence was submitted for review.

On failure, the UI must not falsely report success and should surface the failure through the existing appropriate error path.

Do not introduce an unrelated visual redesign.

### H. Security

Preserve or improve the existing authorization boundary:

```text
Applicant:
    can operate only on own current recovery/application data

Reviewer:
    can access authorized completed verification history

Applicant:
    must not gain access to other applicants' completed Reviewer records
```

Any new history read/write path must enforce the established Reviewer authorization model or a minimal equivalent justified by the inspected code.

Do not broaden administrative permissions.

---

## 4. Required Lifecycle Semantics

The implementation must maintain exactly the authorized lifecycle semantics:

```text
PENDING
   ↓
Reviewer decision
   ↓
VERIFIED or REJECTED completed history record

REJECTED
   ↓
Applicant recovery
   ↓
new/current evidence
   ↓
PENDING
   ↓
Reviewer Queue
   ↓
new Reviewer decision
   ↓
new completed history record
```

Do **not** introduce:

```text
UNDER_REVIEW
```

or any other new lifecycle state.

---

## 5. Compatibility Requirements

Normal non-recovery onboarding must remain unchanged.

The existing Reviewer Queue must continue to identify current `PENDING` applicants.

The existing Reviewer decision endpoint and locked Reviewer surfaces must remain compatible with the new persistent history capability.

Do not modify:

- Driver portal;
- Company portal;
- trip/delivery lifecycle;
- marketplace/claiming;
- AI behavior;
- C-05;
- R-03.

---

## 6. Implementation Stop Conditions

**STOP immediately and return the exact blocker instead of expanding scope** if implementation requires any of the following:

- broad schema redesign;
- generalized audit logging;
- event sourcing;
- full evidence-history/versioning architecture;
- broad RLS redesign;
- generalized transaction framework;
- new lifecycle states or a redesigned lifecycle model;
- authentication changes;
- role-model changes;
- Reviewer authority expansion;
- broad navigation/visual redesign;
- unrelated API-contract changes;
- destructive migration that risks existing completed decisions without a safe preservation plan;
- changes outside the narrow recovery + persistent completed-decision requirement;
- any conflict with the locked Reviewer Blueprint.

If the smallest valid implementation cannot satisfy the requirements without one of these expansions, stop and report the precise dependency for a new governance decision.

---

## 7. Required Validation / Evidence

The implementation is not complete until source/build/runtime evidence demonstrates, at minimum:

### Test A — Existing rejection preservation

```text
Reviewer rejects applicant with reason and evidence
→ completed history record exists
→ reason preserved
→ reviewed_at preserved
→ evidence reference remains resolvable where required
```

### Test B — Recovery

```text
Rejected applicant
→ chooses Re-upload Evidence
→ submits corrected evidence
→ request succeeds
→ current identity becomes PENDING
```

### Test C — Queue re-entry

```text
PENDING recovered applicant
→ appears in Reviewer Queue
```

### Test D — Historical preservation after recovery

```text
Old REJECTED record
→ remains visible in Reviewer History
→ old reason remains visible
→ old decision timestamp remains visible
→ old evidence remains resolvable where required
```

### Test E — Decision separation

```text
Recovered applicant
→ receives a new Reviewer decision
→ new completed history record is created
→ prior rejection record remains unchanged
```

### Test F — Failure consistency

Exercise a controlled failed recovery submission and demonstrate:

```text
No false success message
+ no silently incoherent partial state
```

Use existing project-safe failure testing patterns.

### Test G — Authorization

Demonstrate at minimum:

```text
Reviewer history access without authorization → denied
Non-reviewer history access → denied
Applicant cannot access another applicant's completed history → denied
Applicant recovery remains owner-scoped
```

### Test H — Build / TypeScript

Run the relevant build and TypeScript checks and record evidence.

---

## 8. Evidence Reporting Rules

Create the implementation report at:

```text
03_IMPLEMENTATION/implementation_reports/Chat46_Day19_Node7_Phase1b_Reviewer_Observation2_Recovery_Historical_Decision_Preservation_Implementation_Report.md
```

The report must clearly separate:

```text
VERIFIED
INFERRED
UNKNOWN / BLOCKED
```

Do not mark runtime behavior VERIFIED without runtime evidence.

The report must include:

- exact files/migrations/routes changed;
- exact schema/history structure added;
- authorization/RLS changes;
- recovery write path;
- history read path;
- evidence-link handling;
- build/TypeScript evidence;
- runtime test evidence;
- regression results;
- manual verification items still required from Ayush;
- any stop condition encountered.

---

## 9. Manual Verification Boundary

After implementation and automated/runtime evidence are complete, stop for Ayush manual verification of the live Reviewer + applicant workflow.

Do not declare Reviewer acceptance/lock.

Ayush must verify at minimum:

```text
1. Reject an applicant and confirm rejection reason/evidence in History.
2. Recover as rejected applicant and re-submit corrected evidence.
3. Confirm clear success feedback.
4. Confirm applicant becomes PENDING.
5. Confirm Reviewer Queue shows the applicant again.
6. Confirm old rejection remains in History.
7. Open the old history record and verify the old reason/evidence.
8. Complete a new Reviewer decision.
9. Confirm the new decision is separate from the old rejection.
```

---

## 10. Completion Rule

Implementation may be reported **IMPLEMENTATION COMPLETE** only when the narrow authorized capability is implemented and the evidence above is recorded.

The overall Reviewer feature remains **NOT ACCEPTED** until Ayush manually verifies the live workflow.

---

## 11. Authority Boundary

This handoff authorizes only the work explicitly described here and in the governing governance decision.

It does not supersede:

```text
02_ARCHITECTURE/locked_blueprints/Reviewer_Locked_Blueprint.md
```

It does not reopen unrelated historical architecture decisions.

It does not authorize broad system modernization.

---

## 12. Final Execution Instruction

> **Implement the minimum persistent verification-history capability required to preserve completed Reviewer decisions independently from current applicant state, while correcting rejected-applicant recovery so a successful resubmission moves the current identity to PENDING and re-enters the Reviewer Queue. Preserve prior rejection reason, decision timestamp, and evidence linkage where required. Keep authorization secure, preserve locked Reviewer semantics, avoid `UNDER_REVIEW`, and stop immediately if broader redesign becomes necessary. Build, test, record evidence, and leave final live acceptance to Ayush.**
