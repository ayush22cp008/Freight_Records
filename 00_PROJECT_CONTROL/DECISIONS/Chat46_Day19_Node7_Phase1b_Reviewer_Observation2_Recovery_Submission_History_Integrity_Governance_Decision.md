# Chat46 / Day19 / Node7 / Phase1b
# Reviewer Observation 2 — Recovery Submission / History Integrity Governance Decision

**Project:** Freight — AI Builders Hackathon  
**Node:** Node 7 — AI + Final Integration + Demo  
**Phase:** Phase 1b — Full 3-Portal UI/UX Redesign  
**Day:** Day 19  
**Chat:** Chat46  
**Date:** 2026-09-10  
**Decision owner:** ChatGPT — Architecture / Governance Brain  
**Approval authority:** Ayush  
**Status:** **APPROVED — NARROW RECOVERY SUBMISSION / HISTORY-INTEGRITY CORRECTION AUTHORIZED**

---

## 1. Purpose

This record converts the completed Observation 2 follow-up investigation and Ayush's explicit approval into a narrowly-scoped governance authorization for correcting the rejected-applicant recovery submission failure.

The investigation established that the current recovery implementation is blocked by an RLS-induced partial write and that the resulting duplicate evidence rows cause the Reviewer History detail query to fail. The applicant therefore remains `REJECTED`, does not re-enter the Reviewer Queue, receives no clear successful submission outcome, and the History detail cannot resolve the correct evidence/reason record.

This decision authorizes only the minimum correction required to make the approved recovery flow operational and safe.

This is **not** authorization for a general evidence-history redesign or broad lifecycle rewrite.

---

## 2. Evidence Basis

Primary evidence:

1. `05_DEBUGGING/investigations/Chat46_Day19_Node7_Phase1b_Reviewer_Observation2_Recovery_History_Integrity_Investigation_Report.md`
2. `03_IMPLEMENTATION/implementation_reports/Chat46_Day19_Node7_Phase1b_Reviewer_Observation2_Rejected_Applicant_Recovery_Implementation_Report.md`
3. `00_PROJECT_CONTROL/DECISIONS/Chat46_Day19_Node7_Phase1b_Reviewer_Observation2_Rejected_Applicant_Recovery_Governance_Decision.md`
4. `02_ARCHITECTURE/locked_blueprints/Reviewer_Locked_Blueprint.md`
5. `00_PROJECT_CONTROL/ROADMAP.md`
6. `00_PROJECT_CONTROL/CURRENT_STATUS.md`
7. `00_PROJECT_CONTROL/PROJECT_STATE.md`

The investigation marks as **VERIFIED**:

- the authenticated submission client cannot delete from `onboarding_evidence` under the existing RLS policy;
- the authenticated submission client cannot update `freight_identities` under the existing RLS policy;
- the recovery endpoint therefore partially writes and returns HTTP 500;
- the identity remains `REJECTED`;
- Reviewer Queue correctly filters to `PENDING`, so the applicant does not reappear;
- the old rejection evidence/reason remains in the database;
- a second evidence row is inserted;
- Reviewer History detail uses `.single()` for evidence and therefore fails with multiple rows;
- the UI then falls back to "No reason recorded" / "No evidence document found";
- the current implementation does not provide clear applicant-facing submission-success feedback.

---

## 3. Approved Recovery Correction

Ayush approves the following narrow correction.

### 3.1 Successful recovery submission

The existing applicant recovery path must be made operational for authenticated `REJECTED` users.

A successful corrected-evidence submission must produce a coherent state in which:

```text
REJECTED
   ↓
new corrected evidence accepted
   ↓
PENDING
   ↓
Reviewer Queue eligible
```

The applicant must not remain silently or ambiguously `REJECTED` after a successful recovery submission.

### 3.2 Minimum authorization correction

Authorize the minimum server-side mechanism necessary for the recovery operation to modify the applicant's own identity/evidence records securely.

The implementation may use an existing trusted server/service-role pattern already established in the application, or a narrowly-scoped RLS adjustment, but must preserve applicant ownership checks.

Do not grant broad user-level write privileges to unrelated records.

### 3.3 Consistent write behavior

The correction must prevent the observed partial-write state.

At minimum, the implementation must ensure that the recovery submission does not leave the system in this inconsistent combination:

```text
new evidence inserted
+ identity still REJECTED
```

If the current architecture cannot provide a true database transaction without introducing a larger architectural change, the implementation must use the narrowest safe sequencing/error-handling mechanism available and explicitly report any residual atomicity limitation.

A new general transaction architecture is **not** authorized automatically.

### 3.4 Evidence replacement semantics

The recovery implementation may replace the applicant's current evidence for the purpose of resubmission, but it must not knowingly destroy the information required to render an already-completed Reviewer rejection decision.

The implementation must preserve the existing rejection reason and completed decision evidence sufficiently for the Reviewer History requirement.

Do not introduce a new history table, evidence-history schema, or generalized versioning system unless the current implementation demonstrates that the approved recovery cannot meet the locked History requirement without it. Such a finding is a stop condition requiring a new governance decision.

### 3.5 Reviewer History integrity

The Reviewer History list and selected completed-record detail must remain capable of representing the prior rejected decision with:

- Rejected status;
- persisted rejection reason;
- decision timestamp;
- associated submitted evidence where the existing Blueprint requires it.

The implementation must not solve the Queue issue by corrupting or deleting the completed historical decision.

### 3.6 Reviewer Queue re-entry

After a successful recovery submission, the applicant's identity must be `PENDING` so the existing Reviewer Queue can pick up the new submission through its existing eligibility logic.

Do not redesign the Reviewer Queue query unless required solely to preserve the existing behavior.

### 3.7 Applicant-facing outcome

The applicant must receive a clear success or failure outcome from the evidence resubmission action.

At minimum:

```text
success → clear Pending Verification state / confirmation
failure → explicit error message; do not imply success
```

This may be a frontend-only feedback improvement, provided it accurately reflects the server result.

---

## 4. Historical Decision Semantics

The approved correction must preserve the distinction between:

```text
Prior completed decision
        VS
New pending review submission
```

The prior rejection must remain a completed historical decision rather than being silently converted into a new current state in a way that destroys its historical evidence.

The new submission should create a new reviewable `PENDING` condition without granting Reviewer bypass or applicant approval authority.

If the existing database model cannot represent this distinction safely, implementation must stop and return the exact technical limitation for a new governance decision.

---

## 5. Security / Authorization Requirements

The correction must preserve:

- authenticated applicant ownership checks;
- no cross-user evidence modification;
- no applicant-settable `VERIFIED` state;
- no bypass of Reviewer authorization;
- no broad RLS weakening;
- no new administrative privilege for applicants.

A server/service-role mechanism, if used, must remain behind server-side ownership validation and must not be exposed to the client.

---

## 6. Explicitly Not Authorized

This decision does **not** authorize:

- general evidence-history redesign;
- new evidence tables or generalized audit/history architecture;
- broad RLS redesign;
- broad authorization changes;
- authentication changes;
- role-model changes;
- new persistent review states;
- `UNDER_REVIEW`;
- automated verification;
- Reviewer authority expansion;
- new evidence types or requirements;
- Reviewer navigation or visual redesign;
- Evidence-load gating issue;
- Driver portal changes;
- Company portal changes;
- trip/delivery changes;
- marketplace changes;
- AI behavior changes;
- C-05;
- R-03;
- unrelated API-contract changes.

Any requirement outside this exact scope is a stop condition.

---

## 7. Required Implementation Validation

The implementation handoff must require evidence for all of the following:

### Recovery success

```text
Rejected applicant
→ Re-upload Evidence
→ select corrected evidence
→ submit
→ API succeeds
→ status = PENDING
```

### Applicant result

```text
submission success
→ clear Pending Verification state / success confirmation
```

### Reviewer Queue

```text
PENDING applicant
→ appears in Reviewer Queue
```

### Reviewer History

```text
prior rejected record
→ still readable
→ rejection reason preserved
→ decision timestamp preserved
→ evidence linkage preserved where supported
```

### Security

```text
applicant A
→ can modify only applicant A's recovery data

applicant A
→ cannot modify applicant B

applicant
→ cannot set VERIFIED
```

### Failure handling

A forced/observed submission failure must present an explicit failure state and must not leave a falsely successful UI state.

---

## 8. Implementation Gates

**Gate A — Dedicated implementation handoff**  
Create one implementation prompt based only on this decision.

**Gate B — Implementation**  
Correct recovery submission, state transition, history preservation, and outcome feedback.

**Gate C — Build/test/evidence**  
Run build and relevant tests/checks. Verify RLS/authorization and state behavior.

**Gate D — Antigravity implementation report**  
Record exact changes, tests, runtime findings, and any stop-condition discoveries.

**Gate E — Ayush manual verification**  
Verify the full recovery flow in the running application and Reviewer portal.

**Gate F — Acceptance**  
Observation 2 remains open until manual verification passes.

---

## 9. Stop Conditions

Stop and return to governance if:

- immutable historical rejected evidence cannot be preserved without a schema/history redesign;
- a general-purpose transaction layer is required;
- broad RLS or authorization changes are required;
- the existing evidence model cannot represent current recovery plus historical decision state;
- the implementation would change Reviewer decision authority;
- existing verified applicant behavior is affected outside the recovery path;
- any unrelated protected boundary is crossed.

---

## 10. Final Decision

> **Ayush approves a narrow recovery submission / history-integrity correction.** The project may correct the RLS-induced partial-write failure, make the rejected applicant's resubmission succeed, transition the identity to `PENDING`, preserve the prior rejection decision/history sufficiently for Reviewer History, and provide accurate applicant-facing success/failure feedback. No broad evidence-history redesign is authorized unless implementation proves it unavoidable, in which case work must stop for a new governance decision.

**Ayush approval:** YES  
**Recovery correction:** APPROVED — NARROW  
**REJECTED → PENDING:** AUTHORIZED  
**Prior rejection history preservation:** REQUIRED  
**Applicant ownership/security:** REQUIRED  
**Broad history redesign:** NOT AUTHORIZED  
**Observation 2:** OPEN — IMPLEMENTATION MAY PROCEED

---

## 11. Next Action

Create the dedicated implementation handoff for the **Recovery Submission / History Integrity Correction** using this record as the sole implementation boundary.

Do not combine this work with the separate evidence-load gating or Reviewer navigation/design issues.
