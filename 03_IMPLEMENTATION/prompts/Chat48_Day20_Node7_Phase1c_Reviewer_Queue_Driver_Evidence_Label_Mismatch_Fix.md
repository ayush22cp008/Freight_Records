# Chat48 / Day20 / Node7 / Phase1c
# Reviewer Queue — Driver Evidence Label Mismatch Fix

**Status:** APPROVED FOR IMPLEMENTATION  
**Project:** Freight — AI Builders Hackathon  
**Node:** Node 7 — AI + Final Integration + Demo  
**Phase:** Phase 1c  
**Chat:** Chat48  
**Day:** Day20  
**Executor:** Antigravity  
**Scope:** Correct Reviewer-side Driver evidence label mapping only

## 1. Objective

Fix the confirmed Reviewer evidence-type display mismatch:

```text
DRIVER applicant
actual evidence type = DRIVING_LICENCE
current Reviewer display = GST Document ❌
required display = Driving Licence ✅
```

The stored evidence type must not be changed.

## 2. Authoritative Investigation

Read before implementation:

```text
05_DEBUGGING/investigations/Chat48_Day20_Node7_Phase1c_Reviewer_Queue_Driver_Evidence_Label_Mismatch_Investigation_Report.md
```

The investigation verified that Driver onboarding submits `DRIVING_LICENCE`, while Reviewer-side display logic currently checks for `LICENSE` and falls back to `GST Document` for any other value.

## 3. Required Mapping

Reviewer-facing document labels must use:

```text
DRIVING_LICENCE → Driving Licence
GST             → GST Document
```

Do not create or rename database enum/string values.

Do not modify the onboarding submission value `DRIVING_LICENCE`.

## 4. Affected Reviewer Components

The investigation identified these Reviewer-side components for inspection:

```text
src/app/(authenticated)/reviewer/queue/page.tsx
src/app/(authenticated)/reviewer/verify/[id]/ApplicantVerificationClient.tsx
src/app/(authenticated)/reviewer/history/[id]/EvidenceViewerClient.tsx
```

Inspect all three and correct the stale `LICENSE` mapping wherever it is being used for the evidence-type display.

Do not change unrelated logic.

## 5. Preflight — Mandatory

Before editing, report:

- project root / current working directory;
- source repository;
- branch;
- current commit SHA;
- working-tree status;
- exact affected files inspected;
- current evidence-type mappings found in each affected component.

Stop if the repository boundary is not the expected source repository.

## 6. Minimal Fix

Replace the incorrect Reviewer display check for `LICENSE` with the actual Driver evidence type `DRIVING_LICENCE`.

Preserve the existing Company mapping:

```text
GST → GST Document
```

Do not introduce broad fallback behavior that could hide future evidence-type mistakes. Prefer explicit role/type mapping consistent with the existing application conventions.

Do not modify:

- database schema;
- migrations;
- onboarding submit API;
- onboarding evidence storage;
- Reviewer decision API;
- identity states;
- Reviewer authorization;
- Queue filtering;
- verification lifecycle;
- navigation/styling;
- Driver or Company business logic.

## 7. Regression Preservation

The following must remain unchanged and working:

```text
Fresh signup
→ Complete Onboarding

Evidence submitted
→ Pending Verification

Reviewer Queue
→ correct applicant + correct document label

Reviewer Verify
→ correct evidence type

Reviewer Reject
→ Application Rejected + reason

Re-upload
→ new PENDING evidence

Reviewer re-verification
→ correct current evidence type

Approve
→ Verified

Verification History
→ historical records preserved
```

## 8. Validation Requirements

After implementation:

1. Run the repository's appropriate type-check/static validation.
2. Run `npm run build`.
3. Inspect `git diff`.
4. Confirm only the intended Reviewer evidence-label mappings changed.
5. Confirm no onboarding/database/API/lifecycle files changed unexpectedly.
6. Record exact commands and outcomes.

Do not claim browser verification unless actually performed and documented.

## 9. Manual Verification Handoff for Ayush

Verify:

```text
A. Create/submit a DRIVER application
   → Reviewer Queue displays “Driving Licence”

B. Open Reviewer Verify
   → evidence document/type remains correct

C. Open completed Driver record in History
   → evidence type is displayed correctly

D. Submit/verify a COMPANY application
   → GST still displays as “GST Document”

E. Regression
   → no change to rejection/re-upload/approval lifecycle
```

## 10. Implementation Report

After implementation and validation, create:

```text
03_IMPLEMENTATION/implementation_reports/Chat48_Day20_Node7_Phase1c_Reviewer_Queue_Driver_Evidence_Label_Mismatch_Fix_Implementation_Report.md
```

The report must include:

- preflight state;
- exact files modified;
- old and new evidence-type mappings;
- type-check result;
- `npm run build` result;
- final diff/scope verification;
- manual Ayush verification status.

## 11. Push Boundary

Do not push automatically.

After implementation, validation, and report creation, wait for the standard project push approval and Ayush's explicit permission.

## 12. Completion Definition

```text
Reviewer Driver label corrected
        ↓
Company GST label preserved
        ↓
Type-check passes
        ↓
Production build passes
        ↓
Diff scope confirmed
        ↓
Implementation report saved
        ↓
Ayush manual verification pending
```

This is a narrow presentation-mapping correction. Do not use it as a reason to change the underlying evidence model.