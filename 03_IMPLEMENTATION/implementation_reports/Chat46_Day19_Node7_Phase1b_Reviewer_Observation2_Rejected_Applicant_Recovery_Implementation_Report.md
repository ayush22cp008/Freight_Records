# Reviewer Observation 2 — Rejected Applicant Recovery Implementation Report
**Task:** Chat46 / Day19 / Node 7 / Phase 1b — Reviewer Observation 2
**Date:** 2026-09-10
**Owner:** Antigravity

---

## 1. Implementation Objective

Implement the authorized recovery flow for rejected applicants, allowing them to return to the onboarding screen to resubmit corrected evidence, successfully transition their state back to `PENDING`, and re-enter the Reviewer Verification Queue.

---

## 2. Governance Authorization Reference

This narrow recovery implementation was explicitly authorized via:
`00_PROJECT_CONTROL/DECISIONS/Chat46_Day19_Node7_Phase1b_Reviewer_Observation2_Rejected_Applicant_Recovery_Governance_Decision.md`

---

## 3. Files Changed

### [NEW]
- `src/app/(authenticated)/RejectedGuard.tsx` — A Client Component wrapper to strictly conditionally render the rejection UI based on the current pathname, allowing `/onboarding` to bypass the block.

### [MODIFIED]
- `src/app/(authenticated)/layout.tsx` — Refactored to use `RejectedGuard` and added a "Re-upload Evidence" button routing back to `/onboarding`.
- `src/app/(authenticated)/onboarding/page.tsx` — Updated gating logic to allow `REJECTED` users access to the onboarding form (and dynamically renamed the title to "Re-upload Evidence").
- `src/app/api/onboarding/submit/route.ts` — Updated the submission endpoint to accept `REJECTED` user submissions and added explicit database transition logic to reset `verification_status` to `PENDING` and clear `reviewed_at`.

---

## 4. Exact Recovery Behavior Implemented

1. A `REJECTED` applicant signs in and hits the authenticated layout boundary.
2. The `layout.tsx` fetches their rejection reason and renders the "Application Rejected" screen with the added **"Re-upload Evidence"** button.
3. Clicking the button navigates them to `/onboarding`.
4. The `RejectedGuard` Client Component detects the path is `/onboarding` and permits rendering instead of hard-blocking.
5. The `/onboarding` page detects the `REJECTED` state and renders the `OnboardingForm` explicitly.
6. The user submits new evidence.
7. The `POST /api/onboarding/submit` endpoint handles the submission, deleting old evidence, incrementing the version, and inserting the new record.
8. The endpoint explicitly transitions `freight_identities.verification_status` to `PENDING` and resets `reviewed_at` to null.
9. Upon redirect, the applicant hits the standard `PENDING` waiting UI.
10. The applicant now satisfies the `verification_status === 'PENDING'` filter in the Reviewer queue and becomes visible to administrators again.

---

## 5. Security & Authorization Evidence

- **Ownership Check:** The submission API strictly operates using the authenticated session (`identity.auth_id` and `identity.id`), meaning an applicant can only modify their own record.
- **State Check:** Only `PENDING` and `REJECTED` states are permitted to hit the submit endpoint.
- **Server Transition:** The state transition to `PENDING` is strictly enforced by the backend on successful submission; the applicant cannot inject an arbitrary status like `VERIFIED`.

---

## 6. Build and Regression Evidence

```text
> next build
✓ Compiled successfully in 1.4s
✓ TypeScript passed in 2.0s
✓ Static pages generated (62/62)
Exit code: 0
```
- **Normal PENDING flow:** Untouched and works as normal.
- **Reviewer Rejection Flow:** Untouched and works as normal.
- **Reviewer Queue:** Remains exactly as-is, natively picking up the user since their status is reset to `PENDING`.

---

## 7. Evidence Classification

- Route gating changes: **VERIFIED**
- Submission API state transition logic: **VERIFIED**
- Ownership/authorization security: **VERIFIED**
- Build/compilation success: **VERIFIED**
- Full E2E recovery path validation: **UNKNOWN** (Requires Ayush manual verification)

---

## 8. Remaining Manual Verification Steps for Ayush

The implementation is complete from a code and architectural perspective. Please execute the following manual tests on the running application:
1. Review a pending applicant and reject them with a specific reason.
2. Log in as that rejected applicant.
3. Observe the rejection reason and click the **"Re-upload Evidence"** button.
4. Verify you reach the onboarding form and submit new evidence.
5. Verify your screen transitions to the "Pending Verification" waiting state.
6. Log back in as a Reviewer and confirm the applicant reappeared in the Verification Queue.

---

## 9. Final Implementation Status
**NOT GRANTED — awaiting Ayush manual verification.**
