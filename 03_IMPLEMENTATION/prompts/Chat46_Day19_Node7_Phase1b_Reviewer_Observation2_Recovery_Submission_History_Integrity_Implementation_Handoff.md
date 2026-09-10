# Chat46 — Day 19 — Node 7 — Phase 1b
## Reviewer Observation 2 — Recovery Submission / History Integrity Implementation Handoff

### Implementation Type

**IMPLEMENTATION ONLY — EXPLICITLY AUTHORIZED**

This handoff implements the narrow correction approved by Ayush for the rejected-applicant recovery submission/history-integrity defect.

The governing authorization is:

```text
00_PROJECT_CONTROL/DECISIONS/Chat46_Day19_Node7_Phase1b_Reviewer_Observation2_Recovery_Submission_History_Integrity_Governance_Decision.md
```

Do not expand beyond that decision.

---

## 1. Problem Being Fixed

The current rejected-applicant recovery path has a verified RLS-induced partial-write failure:

```text
REJECTED applicant
      ↓
Recovery submission
      ↓
old evidence DELETE → blocked by RLS
      ↓
new evidence INSERT → succeeds
      ↓
identity UPDATE to PENDING → blocked by RLS
      ↓
API returns 500
      ↓
identity remains REJECTED
      ↓
Reviewer Queue remains empty
      ↓
History detail sees multiple evidence rows
      ↓
.single() fails
      ↓
"No reason recorded"
"No evidence document found"
```

The applicant also receives no clear successful or failed submission outcome.

The previous rejection reason itself was verified as preserved; the History detail cannot resolve it because of the duplicate evidence rows and the `.single()` query failure.

---

## 2. Authorized Target Behavior

Implement this exact recovery behavior:

```text
Rejected applicant
      ↓
Re-upload Evidence
      ↓
Existing onboarding/evidence form
      ↓
Submit corrected evidence
      ↓
Successful server operation
      ↓
verification_status = PENDING
      ↓
Applicant receives clear Pending Verification outcome
      ↓
Reviewer Queue sees the new pending submission
```

At the same time:

```text
Original rejected decision
      ↓
remains readable in Reviewer History
      ↓
rejection reason preserved
      ↓
decision timestamp preserved
      ↓
submitted evidence linkage preserved where existing model supports it
```

---

## 3. Mandatory Source Review Before Modification

Inspect the current repository state before changing code.

Relevant files include:

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

Also inspect the current RLS/migration definitions affecting:

```text
freight_identities
onboarding_evidence
```

Use the current implementation rather than relying on earlier reports.

---

## 4. Approved Implementation Scope

### A. Correct the recovery submission authorization

Make the recovery submission capable of securely updating the applicant's own required records.

The solution may use:

- an existing trusted server/service-role pattern already used by the application, or
- a narrowly-scoped RLS adjustment.

Prefer the smallest change consistent with the existing architecture.

Do not grant broad write access to applicants or unrelated records.

Any server/service-role access must remain server-side and must validate the authenticated applicant identity before mutation.

### B. Prevent the observed partial-write state

The recovery flow must not intentionally leave:

```text
new evidence exists
AND
identity remains REJECTED
```

Implement the narrowest safe write sequencing/error handling available.

A true generalized transaction architecture is **not authorized automatically**.

If a database transaction/RPC is genuinely unavoidable to satisfy correctness, STOP and report the exact reason rather than expanding scope silently.

### C. Preserve the previous completed rejection

Do not destroy the historical information needed for the already-completed Reviewer rejection.

The previous rejection must remain readable with:

- Rejected status;
- rejection reason;
- decision timestamp;
- evidence linkage where the existing model can support it.

Do not create a generalized evidence-history architecture unless the existing model proves incapable of meeting this requirement. Such a finding is a STOP condition.

### D. Support the new pending submission

After successful recovery submission, the current identity must be:

```text
PENDING
```

The new evidence must be associated with the applicant's current reviewable submission.

Reviewer remains the sole verification decision authority.

Do not introduce `UNDER_REVIEW` or any other persistent review state.

### E. Fix duplicate-row/history behavior caused by recovery

The implementation must ensure the recovery path does not leave the History detail in the observed duplicate-evidence failure state.

Do not merely suppress `.single()` errors while allowing corrupted or ambiguous data to remain.

The historical record must remain deterministically resolvable under the existing Reviewer History design.

### F. Accurate applicant submission feedback

On successful submission:

```text
show/redirect to clear Pending Verification state
```

On failure:

```text
show an explicit failure message
```

Never present success if the API failed.

Do not silently swallow an HTTP 500 or equivalent submission failure.

---

## 5. Security Requirements

The implementation must preserve:

```text
Authenticated applicant A
→ can modify only applicant A's own onboarding/recovery data

Applicant A
→ cannot modify applicant B

Applicant
→ cannot set VERIFIED

Applicant
→ cannot bypass Reviewer verification
```

Do not weaken authentication or role rules.

Do not expose service-role credentials/client behavior to browser code.

Do not broaden RLS policies beyond the exact ownership need.

---

## 6. Existing Behavior That Must Remain Unchanged

The following are protected:

- normal `PENDING` onboarding flow;
- normal evidence upload constraints and validation;
- Reviewer Reject/Approve decision semantics;
- Reviewer authorization;
- Reviewer Queue eligibility semantics (`PENDING`);
- Reviewer History list semantics;
- existing decision timestamp behavior;
- Driver portal;
- Company portal;
- authentication model;
- role model;
- trip/delivery workflows;
- marketplace/claiming;
- AI behavior.

---

## 7. Explicitly Out of Scope

Do not implement:

- generalized evidence-history redesign;
- new history tables;
- broad evidence versioning architecture;
- broad RLS redesign;
- generalized transaction framework;
- new persistent review states;
- `UNDER_REVIEW`;
- automated verification;
- applicant self-approval;
- Reviewer authority expansion;
- Reviewer navigation redesign;
- Reviewer visual redesign;
- evidence-load gating issue;
- Driver changes;
- Company changes;
- trip/delivery changes;
- AI changes;
- C-05;
- R-03.

Any newly discovered requirement outside this exact scope is a STOP condition.

---

## 8. Required Validation

### Test A — Normal pending onboarding

Verify existing non-rejected applicant behavior remains unchanged.

### Test B — Rejected applicant recovery

```text
Reject applicant with a concrete reason
→ login as rejected applicant
→ click Re-upload Evidence
→ select corrected evidence
→ submit
```

Expected:

```text
API succeeds
status becomes PENDING
clear Pending Verification outcome
```

### Test C — Reviewer Queue re-entry

As Reviewer:

```text
open Verification Queue
→ applicant appears as PENDING
```

### Test D — Previous History integrity

Open the prior completed rejected record.

Expected:

```text
Rejected
correct rejection reason
correct decision timestamp
submitted evidence available where supported
no "No reason recorded" caused by duplicate evidence
no "No evidence document found" caused by duplicate evidence
```

### Test E — New submission integrity

Verify the newly submitted evidence is available to the Reviewer as part of the new pending review path.

### Test F — Failure handling

Verify a failed submission produces an explicit error and does not claim success.

### Test G — Ownership/security

Verify one applicant cannot mutate another applicant's evidence or identity through this path.

### Test H — Rejection authority preservation

Verify the applicant cannot set `VERIFIED` or otherwise bypass Reviewer decision-making.

---

## 9. Required Build/Test Evidence

Run at minimum:

```text
next build
TypeScript validation
relevant existing automated tests/checks
```

Also provide runtime evidence for:

- successful recovery;
- final `PENDING` state;
- Reviewer Queue re-entry;
- prior rejection History integrity;
- security/authorization behavior;
- failure handling.

Do not classify runtime behavior as VERIFIED without actual evidence.

---

## 10. Required Implementation Report

Create/update only the dedicated implementation report:

```text
03_IMPLEMENTATION/implementation_reports/Chat46_Day19_Node7_Phase1b_Reviewer_Observation2_Recovery_Submission_History_Integrity_Implementation_Report.md
```

The report must contain:

1. Implementation summary
2. Exact files changed
3. Governance authorization reference
4. Recovery flow implemented
5. Authorization/RLS solution
6. Write sequencing/atomicity behavior
7. Historical rejection preservation behavior
8. Reviewer Queue re-entry behavior
9. Applicant success/failure feedback
10. Security evidence
11. Build/test results
12. Runtime evidence
13. Evidence classification (`VERIFIED / INFERRED / UNKNOWN`)
14. Any stop-condition discoveries
15. Final implementation status

Do not modify historical reports.

---

## 11. Mandatory Stop Conditions

STOP and return the issue to ChatGPT/Ayush governance if:

- a new history/evidence schema is required;
- a generalized transaction architecture is required;
- broad RLS changes are required;
- historical rejection evidence cannot be preserved with the current model;
- the existing onboarding model conflicts with the approved recovery behavior;
- Reviewer History cannot remain coherent without architectural redesign;
- any protected lifecycle semantics must change beyond `REJECTED → PENDING`;
- any unrelated portal/system behavior must change.

Do not solve a stop condition by silently expanding scope.

---

## 12. Mandatory Completion Rule

Implementation is **not accepted** merely because the build passes.

The implementation report must return the work for Ayush manual verification.

Final status must remain:

```text
NOT GRANTED — awaiting Ayush manual verification
```

until Ayush verifies the complete flow.

---

## 13. Final Authorization

> Implement only the narrow Recovery Submission / History Integrity correction authorized by the governance decision. Make rejected-applicant resubmission actually succeed, transition to `PENDING`, re-enter Reviewer Queue, preserve the prior rejected decision/history, and provide accurate applicant feedback. Do not broaden the architecture. Stop if the existing model cannot satisfy these requirements without a new architectural decision.
