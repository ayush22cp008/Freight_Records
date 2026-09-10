# Chat46 — Day 19 — Node 7 — Phase 1b
## Reviewer Observation 2 Implementation Handoff — Rejected Applicant Recovery / Evidence Re-submission

### Implementation Type

**TARGETED IMPLEMENTATION — NARROW GOVERNANCE EXCEPTION APPROVED**

This handoff authorizes one specific fix: allow an authenticated applicant whose verification status is `REJECTED` to recover through the existing onboarding/evidence-upload flow, submit corrected evidence, transition to `PENDING`, and become eligible for the existing Reviewer queue again.

This handoff is governed by:

```text
00_PROJECT_CONTROL/DECISIONS/Chat46_Day19_Node7_Phase1b_Reviewer_Observation2_Rejected_Applicant_Recovery_Governance_Decision.md
```

Do not expand beyond that decision.

---

## 1. Verified Problem

The Observation 2 investigation established:

- `src/app/(authenticated)/layout.tsx` returns the rejection UI instead of nested children for `REJECTED` identities;
- `/onboarding` currently requires `PENDING` and redirects otherwise;
- `POST /api/onboarding/submit` currently rejects non-`PENDING` identities;
- the Reviewer queue only retrieves `PENDING` identities;
- there is no existing `REJECTED → PENDING` recovery transition;
- authentication itself is not the blocker.

Therefore, adding only a button is insufficient.

---

## 2. Required Product Behavior

Implement this recovery path:

```text
Applicant is REJECTED
        ↓
Applicant sees Application Rejected + rejection reason
        ↓
Applicant clicks recovery action
        ↓
Existing onboarding/evidence-upload page
        ↓
Applicant uploads corrected evidence
        ↓
Existing submission flow succeeds
        ↓
REJECTED → PENDING
        ↓
Applicant leaves rejected state
        ↓
Reviewer Queue can see applicant again
        ↓
Reviewer makes a new verification decision
```

Preferred recovery action label:

```text
Re-upload Evidence
```

Equivalent wording is acceptable only if it matches the existing design system and clearly communicates recovery.

---

## 3. Authorized Implementation Scope

### A. Rejected-state recovery action

Add a clear applicant-facing recovery action to the existing rejected state.

Requirements:

- visible only to the authenticated rejected applicant;
- navigates to the existing onboarding/evidence-upload surface;
- does not create a new recovery page unless technically unavoidable;
- preserve the existing rejection reason display from Observation 1.

### B. Authenticated layout gating

Modify the rejected-state handling only enough to permit the rejected applicant to reach the existing onboarding flow.

Do not remove authentication or ownership checks.

Do not allow arbitrary authenticated nested routes to bypass existing protections.

Prefer the smallest route-aware exception that safely supports recovery.

### C. Onboarding page gating

Modify the onboarding page gate only enough to permit an authenticated `REJECTED` applicant to use the existing evidence-upload form for recovery.

Do not weaken access for unauthenticated users or other roles.

Do not redesign the onboarding form.

### D. Submission API

Modify `POST /api/onboarding/submit` only enough to support the approved rejected-applicant recovery path.

Preserve:

- authenticated user identity checks;
- applicant ownership checks;
- existing evidence validation;
- existing evidence storage behavior;
- existing submission validation;
- existing protection against another user's record.

Do not create a parallel submission endpoint unless existing architecture makes it unavoidable. If unavoidable, STOP and report before implementing.

### E. Explicit state transition

On successful corrected evidence submission by a `REJECTED` applicant, perform exactly:

```text
REJECTED → PENDING
```

The transition must be server-controlled.

The applicant must not be able to submit an arbitrary target status.

Do not allow:

```text
REJECTED → VERIFIED
REJECTED → arbitrary status
```

If the existing `reviewed_at` field from R-05 is part of the rejected record, clear/reset it only if required to make the new pending review coherent with the existing system.

Do not introduce `UNDER_REVIEW` or another persistent state.

### F. Reviewer queue re-entry

After successful resubmission, the existing Reviewer queue must recognize the applicant through its existing `PENDING` filter.

Do not redesign Reviewer Queue.

Do not alter Reviewer authority or decision semantics.

---

## 4. Evidence Handling Boundary

Reuse the existing onboarding/evidence model and storage flow.

Do not:

- introduce new evidence types;
- redesign evidence schema;
- introduce a new evidence versioning model;
- change evidence requirements;
- modify signed-URL security behavior;
- change Reviewer evidence examination semantics.

Use the existing mechanism for replacing/deleting/inserting onboarding evidence as established by the investigation.

If existing evidence replacement behavior creates a data-integrity concern that cannot be solved within this narrow scope, STOP and report it.

---

## 5. Authorization / Security Requirements

The recovery must be strictly applicant-owned.

Verify that:

```text
Authenticated rejected Applicant A
        ↓
can recover Applicant A only
```

and cannot:

```text
recover Applicant B
modify Applicant B evidence
change another identity status
set their own status directly to VERIFIED
bypass Reviewer verification
```

Do not make broad RLS/security changes.

If a security change broader than the minimum owner-scoped recovery operation is required, STOP and report the boundary rather than expanding the implementation.

---

## 6. Protected Scope

Do NOT modify:

```text
Reviewer Blueprint
Reviewer Queue design
Reviewer navigation
Reviewer visual redesign
Evidence-load gating observation
Shared cross-portal navigation
Authentication architecture
Role model
Trip/delivery workflows
Driver marketplace
Company portal
AI behavior
C-05
R-03
```

Do not combine this implementation with Observation 3 or other manual-verification issues.

---

## 7. Acceptance Criteria

### Recovery entry

```text
REJECTED applicant
→ sees rejection reason
→ sees Re-upload Evidence action
→ action reaches existing onboarding/evidence-upload form
```

### Submission

```text
REJECTED applicant
→ uploads corrected evidence
→ submits successfully
→ existing authentication/ownership checks remain enforced
```

### State

```text
Before submission:  REJECTED
After successful submission: PENDING
```

### Reviewer re-entry

```text
PENDING applicant
→ appears in existing Reviewer queue
```

### Security

```text
Applicant cannot modify another applicant
Applicant cannot self-approve
Applicant cannot bypass Reviewer
```

### Regression

Verify that:

- normal `PENDING` onboarding still works;
- existing `VERIFIED` behavior is unchanged;
- existing `REJECTED` reason display remains intact;
- Reviewer rejection flow remains unchanged;
- Reviewer queue behavior for normal pending applicants remains unchanged.

---

## 8. Required Testing

At minimum, run:

1. TypeScript/build checks.
2. Relevant existing automated tests.
3. Rejected applicant recovery test.
4. Successful evidence resubmission test.
5. `REJECTED → PENDING` state verification.
6. Reviewer queue re-entry verification.
7. Authorization/ownership negative test.
8. Regression test for normal pending onboarding.

If runtime infrastructure is unavailable, clearly classify runtime checks as `UNKNOWN` rather than claiming VERIFIED.

---

## 9. Required Implementation Report

Create/update only this new report:

```text
03_IMPLEMENTATION/implementation_reports/Chat46_Day19_Node7_Phase1b_Reviewer_Observation2_Rejected_Applicant_Recovery_Implementation_Report.md
```

The report must include:

1. Implementation objective
2. Governance authorization reference
3. Files changed
4. Exact recovery behavior implemented
5. Route/layout gating changes
6. Onboarding changes
7. Submission API changes
8. State-transition evidence
9. Reviewer queue re-entry evidence
10. Authorization/security evidence
11. Build/test results
12. Regression results
13. Any migration/database action, if required
14. Evidence classification (`VERIFIED / INFERRED / UNKNOWN`)
15. Scope compliance check
16. Remaining manual verification steps for Ayush
17. Final implementation status

Do not claim manual verification unless Ayush has performed it.

---

## 10. Mandatory Stop Conditions

STOP and return an implementation report without expanding scope if:

- a new persistent lifecycle state is required;
- `REJECTED → PENDING` cannot be implemented safely with the existing model;
- a new table is required;
- broad schema redesign is required;
- broad RLS/security changes are required;
- authentication or role-model changes are required;
- evidence-model redesign is required;
- Reviewer authority must change;
- existing Reviewer behavior must change beyond queue re-entry;
- another protected project area must be modified;
- the implementation conflicts with the locked Reviewer Blueprint;
- any requirement outside the approved governance decision is discovered.

---

## 11. Mandatory Completion Boundary

After implementation, build/test, and evidence recording:

**STOP.**

Do not mark Observation 2 accepted.
Do not modify project closure records.
Do not start Observation 3.

Wait for Ayush manual verification of:

```text
Rejected applicant
→ rejection reason visible
→ Re-upload Evidence
→ existing onboarding page
→ corrected evidence upload
→ successful submission
→ status becomes PENDING
→ rejected screen no longer traps applicant
→ applicant appears in Reviewer Queue
```

Only after Ayush confirms this path works should Observation 2 move to acceptance/closure.
