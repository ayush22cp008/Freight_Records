# Governance Decision — Chat47 / Day 20 / Node 7 / Phase 1c
## Rejected Applicant Recovery — Current Evidence Selection on Onboarding Page

**Status:** APPROVED
**Node:** Node 7 — Reviewer / Driver / Company Verification Recovery
**Phase:** Phase 1c — Rejected Applicant Recovery UX / Current Evidence Resolution
**Decision Type:** Narrow corrective implementation authorization

## 1. Observed Problem

After a rejected Driver or Company applicant re-uploads evidence successfully, the backend transition completes and the applicant becomes `PENDING`. The database legitimately contains multiple `onboarding_evidence` rows: the historical rejected row plus the newly submitted pending row.

The authenticated `/onboarding` page currently queries `onboarding_evidence` by `auth_id` using `.single()` across all evidence rows. When more than one row exists, the query cannot return one row, so the page receives no usable evidence object.

Because the Pending Verification branch depends on the evidence object being present, the page falls through to the generic `Complete Onboarding` UI even though the identity is already `PENDING`. Refreshing/submitting again repeats the same result.

## 2. Verified Runtime / Database Truth

The specific recovered applicant used during the Day 20 live audit showed:

- `verification_status = PENDING`
- `evidence_count = 2`
- one historical `DRIVING_LICENCE` row with `status = REJECTED`
- one newly submitted `DRIVING_LICENCE` row with `status = PENDING`
- exactly one reviewer decision row, the original `REJECTED` decision

The successful re-upload request returned HTTP 200 with `success: true` and therefore did not create a second automatic rejection.

## 3. Root Cause

**VERIFIED:** applicant onboarding page has a cardinality mismatch with the append-only evidence model.

`onboarding_evidence` is a historical evidence log, not a one-row-per-applicant table. The onboarding page incorrectly requires a single row across all historical evidence using `.single()`.

## 4. Decision

Authorize a narrow fix in the authenticated applicant onboarding page so that the current pending evidence is resolved deterministically from the evidence history and the page renders the existing `Pending Verification` state whenever the identity is `PENDING`.

The fix must:

1. Stop relying on `.single()` across all evidence rows for the onboarding-state decision.
2. Resolve the latest relevant `PENDING` evidence row deterministically for the applicant.
3. Preserve the existing lifecycle semantics: `REJECTED -> PENDING` after a valid re-upload.
4. Preserve historical rejected evidence and the historical reviewer decision.
5. Prevent a recovered `PENDING` applicant from falling through to `Complete Onboarding` merely because multiple evidence rows exist.
6. Preserve the existing successful submission flow and do not introduce a new lifecycle state.

## 5. Explicit Non-Goals

This decision does **not** authorize:

- reviewer Queue redesign;
- reviewer Verify current-evidence selection changes already handled by the separate Phase 1b decision;
- reviewer History redesign;
- schema redesign or deletion of historical evidence;
- automatic reviewer decisions;
- new onboarding lifecycle states;
- unrelated Driver or Company portal changes;
- broad UI redesign.

## 6. Acceptance Criteria

A fix is accepted only when all of the following are true:

- A rejected applicant can submit new evidence successfully.
- After that submission, the applicant identity remains `PENDING` and the latest submitted evidence remains `PENDING`.
- The applicant onboarding page shows `Pending Verification`, not `Complete Onboarding`.
- The historical rejected evidence row remains preserved.
- The original rejection decision remains preserved.
- Reviewer Queue still surfaces the recovered applicant.
- Reviewer Verify resolves the latest pending evidence without requiring deletion of historical rows.
- A second submit is not required to make the pending state visible.
- No second reviewer rejection is created by the re-upload flow.

## 7. Governance Conclusion

**APPROVED FOR IMPLEMENTATION** as a narrow Phase 1c corrective fix. The implementation agent may proceed only against this scoped decision and must return an implementation report with build/test evidence and any manual verification requirements.
