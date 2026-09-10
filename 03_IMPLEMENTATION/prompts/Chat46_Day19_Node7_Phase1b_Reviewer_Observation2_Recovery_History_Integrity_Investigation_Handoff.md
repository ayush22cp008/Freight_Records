# Chat46 — Day 19 — Node 7 — Phase 1b
## Reviewer Observation 2 Follow-up Investigation Handoff — Recovery Submission vs Verification History Integrity

### Investigation Type

**INVESTIGATION ONLY — NO FIX AUTHORIZED**

This handoff investigates a new manual-verification regression discovered while testing the approved rejected-applicant recovery flow.

Do not implement a fix during this task.
Do not modify application source code.
Do not modify database schema.
Do not modify APIs or contracts.
Do not change the locked Reviewer Blueprint.
Do not alter the existing recovery implementation.

The purpose is to establish the evidence-backed root cause for why a recovery submission appears to lose the previous rejection reason/evidence in Reviewer History and does not produce an expected pending queue entry.

---

## 1. New Manual Observation

After the approved Observation 2 recovery implementation:

1. The rejected applicant can click **Re-upload Evidence**.
2. The applicant reaches `/onboarding`.
3. The applicant can select a replacement evidence file.
4. After submission, the applicant remains/appears to be on the onboarding flow and the outcome is not obvious from the UI.
5. Reviewer Queue shows **All clear / No pending applications**.
6. Reviewer Verification History contains a record for the same applicant showing **Rejected**.
7. That History record shows **No reason recorded**.
8. That History record shows **No evidence document found for this record**.

Screenshots supplied by Ayush are the primary observation evidence for this follow-up.

This indicates a possible data/history integrity problem in the recovery implementation, not merely a missing notification.

---

## 2. Investigation Question

Answer exactly:

> **What happens to the original rejected decision, rejection reason, submitted evidence, and Reviewer History record when a rejected applicant resubmits evidence, and why does Reviewer Queue remain empty while History shows a rejected record with no reason and no evidence?**

Determine the actual root cause from current source, database, and runtime evidence.

---

## 3. Required Investigation Path

Trace this exact lifecycle:

```text
Original applicant
      ↓
Reviewer rejection
      ↓
REJECTED identity + rejection reason/evidence
      ↓
Applicant clicks Re-upload Evidence
      ↓
Recovery onboarding page
      ↓
New evidence upload
      ↓
POST /api/onboarding/submit
      ↓
Existing evidence deletion/replacement behavior
      ↓
freight_identities state transition
      ↓
reviewed_at handling
      ↓
Reviewer History query
      ↓
Reviewer Queue query
      ↓
Rendered History reason/evidence
```

Identify exactly where the historical decision data is lost, overwritten, detached, or misrepresented.

---

## 4. Mandatory Questions

### A. Original rejected record preservation

Determine whether the recovery submission:

- deletes the same `onboarding_evidence` row that contained the rejected decision's evidence/rejection reason;
- overwrites the row;
- creates a new evidence row/version;
- preserves a separate historical record anywhere;
- or leaves the identity row as the only historical representation.

Do not assume that `onboarding_evidence` is designed as an immutable history store.

### B. Rejection reason preservation

Trace exactly what happens to the original `rejection_reason` after resubmission.

Determine whether it is:

```text
preserved
cleared
deleted with the old evidence row
copied to another record
returned as null by History
or otherwise lost
```

The screenshot showing **No reason recorded** must be reconciled with the actual database/source path.

### C. Historical evidence preservation

Determine why Verification History shows **No evidence document found for this record**.

Trace:

- evidence record identifier used by History;
- whether the old record is deleted during resubmission;
- whether the History record points to the deleted row;
- whether the new evidence is associated with the same identity but no longer with the historical rejected decision;
- whether the history implementation assumes current evidence represents historical evidence.

### D. Reviewer Queue state

Determine why the Reviewer Queue shows no pending application after the user appears to resubmit evidence.

Verify the database state of `freight_identities.verification_status` after submission.

Determine whether the state is actually:

```text
PENDING
REJECTED
other
or failed to update
```

If it is `PENDING`, determine why the queue query does not return it.

If it is still `REJECTED`, determine why the approved transition did not persist.

### E. Submission success / failure visibility

Determine whether the applicant's submission actually succeeds.

Trace:

- upload result;
- `/api/onboarding/submit` response;
- redirect behavior;
- client-side error handling;
- database write sequence;
- whether one write succeeds while another fails.

The user currently cannot clearly tell whether the request and evidence submission completed successfully. Establish whether this is a UI feedback problem, a backend transaction problem, or both.

### F. Atomicity / partial failure

Inspect whether the submission endpoint performs multiple dependent writes without atomic rollback.

Specifically check for a sequence similar to:

```text
Delete old evidence
→ Insert new evidence
→ Update identity to PENDING
```

and determine whether partial failure can leave the system in an inconsistent state.

Do not redesign transactions during investigation; only establish whether a partial-write risk or observed partial-write state exists.

### G. History model semantics

Determine whether the current Reviewer History design represents:

- immutable historical decisions;
- current identity state snapshots;
- current identity joined to current evidence;
- or another model.

This is critical because a recovery transition may legitimately create a new pending application while the original rejected decision must remain readable as historical evidence.

### H. Re-entry semantics

Determine whether the intended recovery behavior should result in:

```text
Old Rejected decision remains immutable in History
AND
New submission creates a fresh PENDING reviewable state
```

or whether the current architecture intentionally reuses one record.

Do not decide a new product model during investigation. Report what the existing Blueprint/governance records and current implementation support, and identify any mismatch.

---

## 5. Evidence Standards

Use:

```text
VERIFIED
INFERRED
UNKNOWN
```

A root cause may be marked VERIFIED only with source/database/runtime evidence.

Specifically separate:

- screenshot evidence;
- source inspection;
- database inspection;
- runtime response evidence;
- inferred behavior.

Do not claim that the old rejection reason was deleted unless the source or database inspection proves it.

---

## 6. Governing Records

Read before investigation:

```text
00_PROJECT_CONTROL/ROADMAP.md
00_PROJECT_CONTROL/CURRENT_STATUS.md
00_PROJECT_CONTROL/PROJECT_STATE.md

00_PROJECT_CONTROL/DECISIONS/Chat46_Day19_Node7_Phase1b_Reviewer_Observation2_Rejected_Applicant_Recovery_Governance_Decision.md

03_IMPLEMENTATION/implementation_reports/Chat46_Day19_Node7_Phase1b_Reviewer_Observation2_Rejected_Applicant_Recovery_Implementation_Report.md

02_ARCHITECTURE/locked_blueprints/Reviewer_Locked_Blueprint.md
```

Also inspect the current source for:

```text
src/app/(authenticated)/layout.tsx
src/app/(authenticated)/RejectedGuard.tsx
src/app/(authenticated)/onboarding/page.tsx
src/app/(authenticated)/onboarding/OnboardingForm.tsx
src/app/api/onboarding/submit/route.ts
src/app/(authenticated)/reviewer/queue/page.tsx
src/app/api/admin/history/route.ts
src/app/(authenticated)/reviewer/history/[id]/page.tsx
```

Use the current repository state, not an earlier implementation snapshot.

---

## 7. Scope Boundary

Investigate only the **post-resubmission data integrity, history integrity, queue re-entry, and submission-result visibility** problem.

Do not investigate or modify:

```text
Reviewer navigation redesign
Reviewer visual redesign
Evidence-load gating
Shared navigation
Driver portal
Company portal
Trip/delivery systems
Marketplace
AI behavior
C-05
R-03
```

Do not fix the issue during this task.

---

## 8. Required Root-Cause Analysis

The report must explicitly distinguish:

### Observation
What Ayush saw in the screenshots.

### Evidence
What source/database/runtime inspection proves.

### Root cause
The specific technical reason for the combination of:

```text
Queue = empty
History = rejected
History reason = missing
History evidence = missing
Applicant outcome = unclear
```

### Contributing factors
Secondary issues such as weak submission feedback or non-atomic writes.

### Ruled-out hypotheses
Plausible explanations that were investigated and disproved.

### Recovery/history compatibility
Assess whether the current recovery implementation is compatible with the existing Reviewer History requirement for immutable completed decisions.

### Recommended fix scope
Recommendation only. Identify whether correction likely requires:

- frontend feedback only;
- submission flow correction;
- evidence/history data-link correction;
- transaction/atomicity correction;
- recovery state-transition correction;
- Reviewer History model correction;
- or a narrowly scoped combination.

Do not implement.

---

## 9. Required Investigation Report

Create:

```text
05_DEBUGGING/investigations/Chat46_Day19_Node7_Phase1b_Reviewer_Observation2_Recovery_History_Integrity_Investigation_Report.md
```

The report must contain:

1. Investigation objective
2. Exact manual observation
3. Environment/runtime tested
4. Original rejection state
5. Recovery submission data flow
6. Evidence preservation/deletion behavior
7. Rejection reason preservation behavior
8. Verification History data path
9. Reviewer Queue data path
10. Post-submission database state
11. Submission success/failure behavior
12. Partial-write / atomicity assessment
13. Authentication/authorization assessment
14. Evidence classification
15. Root cause
16. Contributing factors
17. Ruled-out hypotheses
18. Recovery/history compatibility assessment
19. Recommended fix scope
20. Stop conditions / unresolved questions
21. Final investigation status

---

## 10. Mandatory Stop Point

After creating the report:

**STOP.**

Do not implement a fix.
Do not create an implementation prompt.
Do not alter the recovery implementation.
Do not modify governance records.
Do not modify historical implementation reports.

Wait for ChatGPT/Ayush to review the investigation findings and explicitly decide the next action.
