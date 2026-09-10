# Chat48 / Day20 / Node7 / Phase1c
# Application Rejected Intermediate UI — Antigravity Implementation Instruction

**Status:** APPROVED FOR IMPLEMENTATION  
**Project:** Freight — AI Builders Hackathon  
**Node:** Node 7 — AI + Final Integration + Demo  
**Phase:** Phase 1c  
**Chat:** Chat48  
**Day:** Day20  
**Executor:** Antigravity  
**Implementation scope:** Rejected-applicant intermediate UI/navigation only

## 1. Objective

Add the missing applicant-facing **Application Rejected** intermediate screen after a reviewer rejects an application.

The current rejection and recovery lifecycle is already working. Do not redesign or replace that lifecycle.

The intended applicant flow is:

```text
Reviewer rejects application
        ↓
Application Rejected screen
        ↓
Show rejection reason
        ↓
Show Re-upload Evidence button
        ↓
Click button
        ↓
Existing Re-upload Evidence onboarding form
        ↓
Submit new evidence
        ↓
Pending Verification
```

## 2. Authoritative Current Behavior

The following behavior has already been manually observed and is considered working:

- Reviewer Queue shows submitted applicant.
- Reviewer can open Verify.
- Identity / Role Verification can be confirmed.
- Reviewer can reject with a rejection reason.
- Rejection is preserved in Verification History.
- Applicant reaches the existing `/onboarding` re-upload form.
- Re-upload creates a new pending evidence submission.
- Applicant returns to Pending Verification.
- Reviewer Queue shows the recovered applicant.
- Reviewer can verify and approve.
- Approved applicant reaches the normal application/dashboard flow.

Do not change these behaviors except for inserting the missing rejected-state presentation described below.

## 3. Preflight — Mandatory

Before changing any source file, confirm and report:

- project root;
- current working directory;
- application source repository;
- current branch;
- current commit SHA;
- working-tree status;
- files relevant to the current onboarding/rejection routing.

Stop and report if the repository boundary is not the expected source repository.

## 4. Required UI Behavior

When the authenticated applicant's identity is in `REJECTED` state, the applicant-facing `/onboarding` route should first show a dedicated **Application Rejected** screen rather than immediately presenting the upload form.

The screen must communicate:

```text
Application Rejected

Your application was rejected.

Rejection Reason:
<the actual reviewer-provided rejection reason>

[ Re-upload Evidence ]
```

The exact existing product visual language should be preserved where practical. Do not introduce unrelated styling changes.

The rejection reason must come from the persisted reviewer decision/rejection data already used by the existing rejection/history flow. Do not hard-code a reason and do not create duplicate rejection state.

## 5. Re-upload Navigation

The **Re-upload Evidence** button must open the existing evidence re-upload form/functionality rather than duplicating the upload implementation.

Reuse the current onboarding evidence submission path and existing role-specific document behavior.

The existing form already supports:

- DRIVER → Driving Licence / `DRIVING_LICENCE`
- COMPANY → GST Document / `GST`

Preserve these semantics.

## 6. State and Lifecycle Constraints

Do not change the approved Phase1c lifecycle logic:

```text
PENDING + no PENDING evidence → Complete Onboarding
PENDING + PENDING evidence    → Pending Verification
REJECTED                       → Application Rejected screen
REJECTED + after re-upload     → PENDING + new PENDING evidence → Pending Verification
```

Do not:

- alter the database schema;
- delete historical evidence;
- mutate historical rejection records;
- change Reviewer Queue filtering;
- change Reviewer Verify decision logic;
- change signup behavior;
- change evidence submission API semantics;
- remove the deterministic newest PENDING evidence selection;
- reintroduce the previous `.single()` multi-row evidence problem;
- introduce a new verification lifecycle state.

## 7. Minimal Implementation Preference

Inspect the existing `/onboarding` page and nearby routing/components first.

Prefer the smallest safe implementation that inserts the rejected intermediate presentation while reusing the current upload form.

A reasonable implementation may involve a small UI state/gating addition and a reusable route/query or local interaction, but Antigravity must inspect the existing code and choose the smallest architecture-consistent solution.

Do not refactor unrelated onboarding code.

## 8. Rejection Reason Data Requirement

Before implementation, identify exactly where the current rejected reason is persisted and where the Reviewer History/Rejected view already reads it.

Reuse that existing source of truth.

If the applicant-facing authenticated route cannot legally read the existing rejection reason because of current authorization/RLS boundaries, do not bypass security or expose broader data. Stop and report the evidence and proposed minimal authorized change before expanding scope.

## 9. Build / Test Requirements

After implementation:

1. run the repository's appropriate TypeScript/static validation;
2. run the repository build command;
3. report exact commands and outcomes;
4. inspect git diff;
5. confirm the change is limited to the rejected-state UI/navigation scope;
6. report any unexpected file changes.

Do not claim browser/manual verification unless actually performed by Antigravity and reproducibly documented. Ayush remains the final manual verifier.

## 10. Mandatory Manual Verification Handoff for Ayush

Provide a concise checklist with these cases:

```text
A. Reviewer rejects DRIVER
   → applicant sees Application Rejected screen
   → rejection reason is shown correctly
   → Re-upload Evidence button is visible

B. Reviewer rejects COMPANY
   → same checks

C. Click Re-upload Evidence
   → existing Re-upload Evidence form opens
   → correct role-specific document label remains

D. Submit replacement evidence
   → applicant reaches Pending Verification
   → current evidence type is shown

E. Reviewer Queue
   → recovered applicant appears once with current PENDING evidence

F. Verification History
   → original rejected decision/reason remains preserved
   → new recovery submission does not erase historical rejection

G. Regression
   → fresh signup still shows Complete Onboarding
   → normal PENDING-with-evidence flow still shows Pending Verification
```

All manual browser checks remain `PENDING` unless independently documented by Ayush.

## 11. Implementation Report

After implementation and validation, create:

```text
03_IMPLEMENTATION/implementation_reports/Chat48_Day20_Node7_Phase1c_Application_Rejected_Intermediate_UI_Implementation_Report.md
```

The report must include:

- preflight repository/branch/commit/working-tree evidence;
- exact source files changed;
- how the rejection reason was obtained;
- exact rejected-state UI/routing behavior implemented;
- build/type-check commands and outcomes;
- diff/scope verification;
- any security/authorization consideration discovered;
- explicit statement that manual Ayush verification remains pending unless documented.

## 12. Push Boundary

Do not push automatically.

After implementation, build/test, and report creation, wait for the project-approved commit/push step and Ayush's explicit permission to push.

## 13. Completion Definition

```text
Rejected-state intermediate UI implemented
        ↓
Existing re-upload flow reused
        ↓
Rejection reason preserved and displayed
        ↓
Build / type-check passes
        ↓
Diff scope confirmed
        ↓
Implementation report saved
        ↓
Ayush manual verification pending
```

This instruction is intentionally narrow: fix the missing **Application Rejected** applicant-facing screen and its transition to the existing re-upload form, while preserving the already verified onboarding/reviewer lifecycle.