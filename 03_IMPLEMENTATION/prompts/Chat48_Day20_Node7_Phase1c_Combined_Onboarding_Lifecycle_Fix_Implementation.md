# Chat48 / Day20 / Node7 / Phase1c
# Combined Onboarding Lifecycle Fix — Antigravity Implementation Instruction

**Status:** APPROVED FOR IMPLEMENTATION
**Project:** Freight — AI Builders Hackathon
**Node:** Node 7 — AI + Final Integration + Demo
**Phase:** Phase 1c
**Chat:** Chat48
**Day:** Day20
**Executor:** Antigravity
**Reasoning / governance owner:** ChatGPT
**Implementation scope:** Minimal onboarding page state-gate correction only

## 1. Authoritative Inputs

Before implementation, read and use these Records as the governing inputs:

```text
05_DEBUGGING/investigations/Chat48_Day20_Node7_Phase1c_Combined_Onboarding_Lifecycle_Investigation_Report.md
00_PROJECT_CONTROL/DECISIONS/Chat47_Day20_Node7_Phase1c_Rejected_Applicant_Current_Evidence_Onboarding_Governance_Decision.md
03_IMPLEMENTATION/implementation_reports/Chat47_Day20_Node7_Phase1c_Rejected_Applicant_Current_Evidence_Onboarding_Implementation_Report.md
00_PROJECT_CONTROL/CHECKPOINTS/Chat47_Day20_Node7_Phase1c_Onboarding_Fix_Checkpoint.md
```

The combined investigation concluded that fresh signup creates a `PENDING` identity before evidence exists, while the previous Phase1c change made the `/onboarding` page treat every `PENDING` identity as already submitted. The approved correction is to distinguish `PENDING` with no evidence from `PENDING` with a current pending evidence row.

## 2. Preflight — Mandatory

Before changing any file, confirm and report:

- project root;
- current working directory;
- application source repository;
- current branch;
- current commit SHA;
- working-tree status;
- exact target file path.

Stop and report if any boundary does not match the expected project/source repository.

## 3. Allowed Source Change

Modify only:

```text
src/app/(authenticated)/onboarding/page.tsx
```

Do not modify the database schema, migrations, signup API, evidence submission API, Reviewer Queue, Reviewer Verify, authentication, RLS/security, Driver portal, Company portal, or any other source file unless a build/test failure proves a strictly necessary compatibility issue. Any such unexpected requirement must be reported before expanding scope.

## 4. Required Behavior

Preserve the already-working deterministic current-evidence query introduced by Phase1c:

- scope by authenticated `auth_id`;
- select only `status = PENDING` evidence;
- order by `created_at DESC`;
- take the newest single row deterministically;
- safely handle no matching row.

Change only the Pending Verification rendering condition so that it requires both:

```text
identity.verification_status === 'PENDING'
AND
current PENDING evidence exists
```

The intended behavior is:

```text
Fresh signup
PENDING + no PENDING evidence
→ Complete Onboarding / evidence upload form

Fresh applicant after evidence submission
PENDING + PENDING evidence exists
→ Pending Verification

Rejected applicant
REJECTED
→ Re-upload Evidence / existing rejected recovery flow

Rejected applicant after successful re-upload
PENDING + new PENDING evidence exists
→ Pending Verification
```

Do not introduce a new lifecycle state.

## 5. Critical Preservation Rules

The existing Phase1c recovery fix must remain intact.

Specifically:

- do not restore the old applicant-wide `.single()` evidence query;
- do not select the historical `REJECTED` evidence row when a newer `PENDING` row exists;
- do not delete or mutate historical evidence;
- do not alter the rejected-applicant re-upload API behavior;
- do not change Reviewer Queue filtering logic;
- do not change Reviewer Verify behavior;
- do not change role semantics for DRIVER or COMPANY.

## 6. Build / Test Requirements

After the change:

1. run the appropriate TypeScript/static validation used by the repository;
2. run the repository build command;
3. report exact commands and results;
4. inspect git diff and confirm only the authorized source file changed;
5. report any unexpected source or generated-file changes.

Do not claim manual browser verification. Ayush performs manual UI verification.

## 7. Required Manual Verification Handoff for Ayush

After successful build/test, provide a concise test matrix for Ayush covering at minimum:

```text
A. Fresh DRIVER signup
   → Complete Onboarding appears
   → evidence upload is available

B. Fresh COMPANY signup
   → Complete Onboarding appears
   → company evidence upload is available

C. DRIVER submits evidence
   → Pending Verification appears
   → Evidence Type is populated
   → Reviewer Queue shows applicant

D. COMPANY submits evidence
   → Pending Verification appears
   → Evidence Type is populated
   → Reviewer Queue shows applicant

E. Rejected DRIVER recovery
   → Re-upload Evidence remains available for REJECTED state
   → after re-upload: Pending Verification
   → historical rejected evidence remains preserved

F. Rejected COMPANY recovery
   → same preservation checks

G. Reviewer Queue / Verify
   → recovered/current PENDING applicant is visible
   → newest PENDING evidence is selected

H. Regression sanity
   → previously verified Driver and Company flows are unchanged
```

Do not mark these manual checks VERIFIED unless Ayush explicitly confirms them.

## 8. Postflight / Implementation Report

Create the Antigravity implementation report in:

```text
03_IMPLEMENTATION/implementation_reports/Chat48_Day20_Node7_Phase1c_Combined_Onboarding_Lifecycle_Fix_Implementation_Report.md
```

The report must include:

- preflight source/branch/commit/working-tree state;
- exact source file modified;
- concise description of the implemented logic;
- exact build/type-check commands and outcomes;
- git diff/file-change scope;
- whether any unexpected changes occurred;
- explicit statement that no manual UI verification was claimed;
- manual verification checklist/results as `PENDING` unless independently documented by Ayush.

## 9. Push Boundary

Do not push automatically.

After implementation and build/test, wait for the separate project-approved commit/push instruction and Ayush's explicit permission to push.

## 10. Completion Definition

Implementation is complete only when:

```text
Correct source change applied
        ↓
Build / type-check passes
        ↓
Diff scope confirmed
        ↓
Implementation report saved
        ↓
Ayush manual verification pending
```

This instruction authorizes implementation only for the minimal onboarding state-gate correction established by the Chat48 investigation and approved decision.