# Chat46 / Day19 / Node7 / Phase1b
# Reviewer Observation 2 — Rejected Applicant Recovery Governance Decision

**Project:** Freight — AI Builders Hackathon  
**Node:** Node 7 — AI + Final Integration + Demo  
**Phase:** Phase 1b — Full 3-Portal UI/UX Redesign  
**Day:** Day 19  
**Chat:** Chat46  
**Date:** 2026-09-10  
**Decision owner:** ChatGPT — Architecture / Governance Brain  
**Approval authority:** Ayush  
**Status:** **APPROVED — NARROW REJECTED-APPLICANT RECOVERY EXCEPTION AUTHORIZED**

---

## 1. Purpose

This record converts the completed Observation 2 investigation and Ayush's explicit approval into a narrowly-scoped governance authorization for implementing a recovery path for applicants whose verification status is `REJECTED`.

The investigation established that the current system does not support recovery through the existing onboarding flow. The problem is not a missing UI button alone; it is a coordinated routing, page-gating, submission, and state-transition limitation.

This record authorizes only the minimum changes necessary to allow a rejected applicant to correct evidence and re-enter the existing verification pipeline.

This is **not** a blanket reopening of lifecycle architecture.

---

## 2. Evidence Basis

The decision is based on:

1. `05_DEBUGGING/investigations/Chat46_Day19_Node7_Phase1b_Reviewer_Observation2_Rejected_Applicant_Recovery_Investigation_Report.md`
2. `03_IMPLEMENTATION/implementation_reports/Chat46_Day19_Node7_Phase1b_Reviewer_Observation1_Rejection_Reason_Applicant_Visibility_Implementation_Report.md`
3. `00_PROJECT_CONTROL/ROADMAP.md`
4. `00_PROJECT_CONTROL/CURRENT_STATUS.md`
5. `00_PROJECT_CONTROL/PROJECT_STATE.md`
6. `02_ARCHITECTURE/locked_blueprints/Reviewer_Locked_Blueprint.md`
7. The existing Phase 1b protected-boundary rules and prior Reviewer R-05 governance decision.

The Observation 2 investigation reports the following as **VERIFIED**:

- `layout.tsx` blocks rejected users from rendering nested authenticated routes.
- `/onboarding` separately requires `PENDING` status.
- `POST /api/onboarding/submit` separately requires `PENDING` status.
- the Reviewer queue only admits `PENDING` identities.
- no existing `REJECTED → PENDING` recovery transition exists.
- authentication itself is not the blocker.

Therefore, a button-only change would not solve the actual problem.

---

## 3. Problem Being Authorized for Resolution

Current behavior:

```text
Reviewer rejection
      ↓
REJECTED
      ↓
Applicant remains authenticated
      ↓
Rejected screen blocks authenticated route tree
      ↓
No onboarding access
      ↓
No evidence resubmission
      ↓
No REJECTED → PENDING transition
      ↓
No Reviewer queue re-entry
```

Required recovery behavior:

```text
Reviewer rejection
      ↓
REJECTED
      ↓
Applicant sees reason
      ↓
Applicant chooses recovery action
      ↓
Existing evidence-upload/onboarding surface
      ↓
Applicant uploads corrected evidence
      ↓
Submission executes authorized recovery transition
      ↓
PENDING
      ↓
Reviewer queue eligible again
```

The exact implementation details must follow the existing system rather than introduce a second onboarding architecture.

---

## 4. Approved Scope

Ayush explicitly approves the following narrow exception.

### 4.1 Rejected applicant recovery entry

Authorize a recovery action from the applicant-facing rejected state that takes the applicant to the existing evidence-upload/onboarding flow.

The action must be available only to the currently authenticated rejected applicant for their own application.

Preferred product intent:

```text
[ Re-upload Evidence ]
```

The final label may remain equivalent if implementation consistency requires it.

### 4.2 Rejected-user access to existing onboarding

Authorize the minimum routing/page-gating change required for a `REJECTED` applicant to access the existing onboarding/evidence-upload form for the purpose of correcting and resubmitting evidence.

Do not create a separate recovery page unless existing architecture proves the current onboarding page cannot safely support the flow.

### 4.3 Rejected-user evidence resubmission

Authorize the minimum submission-path change required for a `REJECTED` applicant to submit corrected evidence through the existing onboarding mechanism.

The implementation must preserve existing authentication and applicant ownership checks.

### 4.4 Explicit recovery state transition

Authorize one explicit recovery transition:

```text
REJECTED → PENDING
```

This transition must occur only when the applicant successfully submits a new/replacement evidence package through the approved recovery path.

Do not introduce:

- `UNDER_REVIEW`;
- a second review state;
- a separate recovery lifecycle;
- automatic approval;
- direct `REJECTED → VERIFIED` behavior.

### 4.5 Review eligibility after successful resubmission

Authorize the minimum behavior necessary for the successfully resubmitted applicant to satisfy the existing Reviewer queue's `PENDING` eligibility and return to normal Reviewer processing.

The Reviewer remains the decision authority.

### 4.6 Existing rejection metadata handling

The implementation may clear or supersede applicant-specific rejection metadata such as `reviewed_at` when moving the application back to `PENDING`, but only as required to keep the existing lifecycle coherent.

The prior rejection reason must not be converted into a new unrelated field or new review model.

If the implementation needs additional metadata handling beyond this narrow requirement, stop and request a new decision.

---

## 5. Security / Authorization Boundary

The recovery flow must preserve these boundaries:

- only the authenticated applicant can initiate recovery for their own identity;
- the applicant cannot modify another user's onboarding/evidence record;
- the applicant cannot directly set `VERIFIED`;
- the applicant cannot bypass Reviewer verification;
- the Reviewer queue remains the source of verification decisions;
- no new broad administrative permission is created.

Do not weaken existing authentication, authorization, RLS, or ownership checks merely to make the UI reachable.

If a new RLS/security change is required beyond the minimum owner-scoped recovery operation, stop and return the finding for governance review.

---

## 6. Explicitly Not Authorized

This decision does **not** authorize:

- redesign of the overall verification lifecycle;
- introduction of `UNDER_REVIEW` or any new persistent review state;
- automated re-verification;
- applicant self-approval;
- Reviewer authority expansion;
- new Reviewer workflows;
- new evidence types or evidence requirements;
- evidence-model redesign;
- new tables unless separately proven unavoidable;
- broad RLS/security redesign;
- broad authentication changes;
- role-model changes;
- trip/delivery changes;
- driver marketplace changes;
- Company portal changes;
- AI behavior changes;
- C-05 changes;
- R-03 changes;
- unrelated API-contract changes;
- Reviewer navigation or visual redesign;
- Evidence-load gating issue from the separate manual observation.

Any requirement outside the exact recovery scope above is a stop condition.

---

## 7. Implementation Gates

The work must proceed through the following gates:

**Gate A — Implementation handoff**  
Create a dedicated implementation prompt containing only the approved recovery scope.

**Gate B — Implementation**  
Implement the recovery action, minimum route/page access, submission support, and `REJECTED → PENDING` transition.

**Gate C — Build/test/evidence**  
Run build and relevant tests/checks. Verify authorization and state transition behavior.

**Gate D — Antigravity implementation report**  
Record exact files, behavior, tests, and any newly discovered boundary issues in the Records repository.

**Gate E — Ayush manual verification**  
Manually test the rejected applicant recovery path from the actual production-facing application:

```text
Rejected applicant
→ sees rejection reason
→ clicks recovery action
→ reaches existing onboarding/evidence upload
→ submits corrected evidence
→ status becomes PENDING
→ rejected state no longer traps the user
→ Reviewer queue can receive the applicant again
```

**Gate F — Acceptance / closure**  
Only after manual verification passes may Observation 2 be marked accepted/closed.

---

## 8. Stop Conditions

Antigravity must stop and report rather than expand scope if any of the following occurs:

- `REJECTED → PENDING` cannot be implemented without a broader lifecycle redesign;
- an additional database model is required beyond the existing identity/evidence records;
- broader RLS or authorization changes are required;
- existing onboarding semantics conflict with this recovery path;
- the existing evidence model cannot safely represent replacement evidence;
- the Reviewer queue requires unrelated redesign;
- the implementation would alter Verified applicant behavior;
- the implementation changes Reviewer authority or decision semantics;
- any protected Phase 1b boundary would be crossed beyond this approved exception.

---

## 9. Final Decision

> **Ayush approves a narrow rejected-applicant recovery exception.** The project may implement the minimum changes needed for an authenticated `REJECTED` applicant to access the existing evidence-upload/onboarding flow, submit corrected evidence, transition explicitly from `REJECTED → PENDING`, and re-enter the existing Reviewer queue. No broader lifecycle redesign or unrelated backend/security changes are authorized.

**Ayush approval:** YES  
**Recovery exception:** APPROVED — NARROW  
**Authorized transition:** `REJECTED → PENDING`  
**Reviewer authority:** PRESERVED  
**Existing onboarding/evidence flow:** REUSE REQUIRED  
**Broader lifecycle redesign:** NOT AUTHORIZED  
**Observation 2:** GOVERNANCE APPROVED — IMPLEMENTATION MAY PROCEED

---

## 10. Next Action

Create the dedicated implementation handoff for **Reviewer Observation 2 — Rejected Applicant Recovery**, using this decision as the sole authorization boundary.

Do not combine this work with Observation 3 (evidence-load gating) or Observation 4/other navigation/design issues.
