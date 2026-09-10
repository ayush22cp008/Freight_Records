# Chat46 — Day 19 — Node 7 — Phase 1b
## Reviewer Observation 2 Investigation Handoff — Rejected Applicant Recovery / Evidence Re-submission Access

### Investigation Type

**INVESTIGATION ONLY — NO FIX AUTHORIZED**

This handoff investigates one specific manual-verification observation in the Reviewer / applicant onboarding workflow.

Do not implement a fix during this task.
Do not modify application source code.
Do not modify database schema.
Do not modify APIs or contracts.
Do not change the locked Reviewer Blueprint.
Do not change onboarding state semantics.
Do not add the recovery button during this investigation.

The purpose is to establish the evidence-backed **root cause** first. A separate implementation prompt will be created only after the investigation report is reviewed and a fix decision is explicitly made.

---

## 1. Observation

During Ayush's manual verification of the Reviewer rejection flow, an applicant was rejected and the rejection reason was correctly shown on the applicant-facing screen after the separate Observation 1 change.

However, the rejected applicant remains authenticated and is effectively trapped on the rejected state because the current authenticated application shell prevents the user from reaching the evidence-upload/onboarding surface again.

Observed behavior:

```text
Reviewer
  ↓
Reject Applicant
  ↓
Reason persisted
  ↓
Applicant signs in / remains signed in
  ↓
Application Rejected screen
  ↓
Reason visible
  ↓
NO RECOVERY / RE-UPLOAD ACTION
  ↓
Applicant cannot naturally return to the evidence-upload page
```

Ayush's expected behavior is:

```text
Application Rejected
Reason: <persisted rejection reason>

[ Re-upload Evidence ]
        ↓
Applicant reaches the existing evidence-upload/onboarding page
        ↓
Applicant can replace/correct the rejected evidence
        ↓
Applicant submits the new evidence through the existing onboarding flow
        ↓
Existing verification/review workflow can continue
```

Important: the investigation must establish whether this is only a navigation/UI omission or whether the authenticated rejection gate, onboarding submission flow, or review-state transition also prevents recovery.

Do not assume that adding a button to `/onboarding` is sufficient.

---

## 2. Investigation Question

Answer exactly:

> **Why is a rejected authenticated applicant unable to return to the existing evidence-upload/onboarding page and resubmit corrected evidence?**

Determine the actual root cause from current source and runtime evidence.

---

## 3. Required Investigation Path

Trace the complete rejected-applicant recovery path:

```text
Reviewer Reject
      ↓
verification_status = REJECTED
      ↓
Authenticated applicant request
      ↓
Current authenticated layout / route guard
      ↓
Rejected-state rendering
      ↓
Attempt to reach existing onboarding/evidence-upload route
      ↓
Onboarding page / component
      ↓
Evidence upload
      ↓
POST /api/onboarding/submit (or current submission path)
      ↓
Database persistence / status transition
      ↓
Reviewer queue eligibility
```

The investigation must establish every point where recovery is allowed, blocked, redirected, or silently prevented.

---

## 4. Questions to Answer

### A. Rejected-state gate

Inspect the current authenticated shell and determine:

- where `verification_status === 'REJECTED'` is detected;
- whether the current rejection UI is rendered from `src/app/(authenticated)/layout.tsx` or another guard;
- what routes are reachable while rejected;
- whether `/onboarding` is explicitly excluded, implicitly blocked, or never considered;
- whether the current Observation 1 implementation changed this control path.

Establish the actual route behavior rather than inferring it from the screenshot.

### B. Existing onboarding/evidence-upload surface

Identify the existing applicant evidence-upload page/component and its route.

Expected starting point includes:

```text
src/app/(authenticated)/onboarding/OnboardingForm.tsx
```

But verify the current repository structure rather than assuming this path is still authoritative.

Determine:

- whether the page is still reachable for authenticated users;
- whether it requires a specific verification state;
- whether rejected users are intentionally prevented from entering it;
- whether the page supports selecting and uploading replacement evidence.

### C. Upload and submission flow

Trace the existing evidence submission flow end-to-end.

Determine:

- upload route/API;
- submit route/API;
- authenticated identity used for submission;
- database records updated;
- existing fields used for evidence metadata and status;
- whether a rejected applicant can overwrite or replace the existing rejected evidence;
- whether an existing `rejection_reason` is preserved, cleared, or superseded;
- whether any new evidence versioning behavior already exists.

Do not design a new evidence model during this investigation.

### D. State transition after resubmission

This is mandatory.

Determine what happens when an applicant who is currently `REJECTED` submits new evidence through the existing onboarding flow.

Establish whether the current implementation:

```text
REJECTED → PENDING
REJECTED → VERIFIED
REJECTED → REJECTED
REJECTED → no status change
REJECTED → blocked/error
```

Do not invent or change the transition.

If no existing supported transition exists, record that as a product/backend dependency rather than implementing one during investigation.

### E. Reviewer queue eligibility after resubmission

Determine whether a resubmitted applicant would become visible to the existing Reviewer queue.

Verify:

- query filters;
- status conditions;
- evidence conditions;
- whether the same applicant identity can re-enter the pending queue;
- whether a new Reviewer decision can be made for the resubmission.

### F. Authorization / privacy boundary

Determine whether allowing the rejected applicant to access the existing onboarding/evidence-upload surface is compatible with the existing authorization model.

Verify that an applicant can only modify or submit evidence for their own identity/onboarding record.

Do not introduce broader RLS/auth changes unless the investigation proves they are necessary.

### G. User-account authentication state

The screenshots show the rejected user remains a valid authenticated Supabase user.

Verify whether the problem is purely application routing/authorization after authentication rather than account deletion, sign-out, or disabled credentials.

Do not infer that the Supabase authentication account should be deleted or recreated.

---

## 5. Current Manual Evidence

Use the screenshots supplied by Ayush as observation evidence, but do not treat screenshots as proof of the technical root cause.

Observed:

```text
1. Applicant is authenticated.
2. Applicant reaches Application Rejected screen.
3. Rejection reason is visible.
4. No action is available to return to evidence upload.
5. Supabase Auth still contains the applicant user.
```

The fourth item is the primary UX observation. The investigation must determine whether the missing action is only a UI omission or whether deeper workflow/state constraints also exist.

---

## 6. Governing Project Records

Read the relevant records before investigation:

```text
00_PROJECT_CONTROL/ROADMAP.md
00_PROJECT_CONTROL/CURRENT_STATUS.md
00_PROJECT_CONTROL/PROJECT_STATE.md

02_ARCHITECTURE/locked_blueprints/Reviewer_Locked_Blueprint.md

00_PROJECT_CONTROL/DECISIONS/Chat46_Day19_Node7_Phase1b_Reviewer_R05_Final_Governance_Decision.md
00_PROJECT_CONTROL/DECISIONS/Chat46_Day19_Node7_Phase1b_Reviewer_R05_Final_Readiness_Decision.md

03_IMPLEMENTATION/implementation_reports/Chat46_Day19_Node7_Phase1b_Reviewer_Frontend_Implementation_Report.md
03_IMPLEMENTATION/implementation_reports/Chat46_Day19_Node7_Phase1b_Reviewer_Observation1_Rejection_Reason_Applicant_Visibility_Implementation_Report.md
```

Also inspect the currently implemented onboarding/evidence-upload code and any existing relevant investigation/report records.

These records define context and boundaries. They do not determine the technical root cause in advance.

---

## 7. Scope Boundary

This investigation is limited to **recovery access for a rejected authenticated applicant and the existing evidence re-submission path**.

Do not investigate or modify:

```text
Reviewer navigation redesign
Reviewer visual redesign
Evidence-load gating (Observation 3 / separate issue)
Shared navigation changes
Authentication redesign
Role model redesign
Trip/delivery systems
Driver marketplace
Company trip workflows
AI behavior
C-05
R-03
```

Do not redesign the evidence model, Reviewer decision model, or lifecycle semantics during this investigation.

---

## 8. Evidence Standards

Use the project's evidence classification:

```text
VERIFIED
INFERRED
UNKNOWN
```

A root cause may be marked **VERIFIED** only when supported by concrete source/runtime evidence.

Distinguish clearly between:

- what the screenshot demonstrates;
- what source inspection demonstrates;
- what runtime testing demonstrates;
- what remains inferred or unknown.

---

## 9. Required Root-Cause Analysis

The report must explicitly distinguish:

### Observation
What Ayush saw.

### Evidence
What source/database/runtime inspection proves.

### Root cause
The specific technical reason the rejected applicant cannot return to the evidence-upload/onboarding flow.

### Contributing factors
Any secondary issues that help produce the trap, such as a route guard or unsupported resubmission state transition.

### Not the root cause
Any plausible hypotheses that were investigated and ruled out.

### Recovery feasibility
State whether the existing system already supports:

- navigation back to onboarding;
- evidence replacement;
- resubmission;
- transition back to review eligibility.

If one or more of these are not currently supported, identify the exact boundary/dependency.

### Recommended fix scope
Recommendation only. Determine whether the eventual fix appears to be:

- frontend recovery/navigation only;
- authenticated layout/route-gating adjustment;
- onboarding form adjustment;
- submission/state-transition adjustment;
- a combination of the above;
- or another narrowly scoped change requiring explicit governance approval.

Do not implement the fix yet.

---

## 10. Required Investigation Report

Create a new report:

```text
05_DEBUGGING/investigations/Chat46_Day19_Node7_Phase1b_Reviewer_Observation2_Rejected_Applicant_Recovery_Investigation_Report.md
```

The report must contain:

1. Investigation objective
2. Exact observation
3. Environment tested
4. Current rejected-state route behavior
5. Existing onboarding/evidence-upload surface
6. Evidence upload and submission flow
7. Rejected-applicant resubmission behavior
8. Database/state transition evidence
9. Reviewer queue re-entry evidence
10. Authentication/authorization assessment
11. Evidence classification (`VERIFIED / INFERRED / UNKNOWN`)
12. Root cause
13. Contributing factors
14. Ruled-out hypotheses
15. Recovery feasibility assessment
16. Recommended fix scope (recommendation only; no implementation)
17. Stop conditions / unresolved questions
18. Final investigation status

---

## 11. Mandatory Stop Point

After the investigation is complete and the report is created:

**STOP.**

Do not fix anything.
Do not create an implementation prompt.
Do not modify the project architecture.
Do not modify `CURRENT_STATUS.md`, `PROJECT_STATE.md`, or `ROADMAP.md` solely because the investigation exists.
Do not change the Observation 1 implementation report.

Wait for ChatGPT/Ayush to review the investigation findings and decide whether a separate implementation prompt should be created.
