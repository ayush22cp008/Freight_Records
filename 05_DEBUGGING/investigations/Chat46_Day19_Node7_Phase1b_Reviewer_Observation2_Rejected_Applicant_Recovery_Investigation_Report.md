# Reviewer Observation 2 — Rejected Applicant Recovery Investigation Report

## 1. Investigation Objective
Determine the root cause preventing an authenticated, rejected applicant from navigating back to the evidence-upload page to resubmit corrected evidence, and establish whether this is a purely UI-level issue or a deeper state/workflow limitation.

## 2. Exact Observation
As reported by Ayush:
- An applicant is rejected by a Reviewer.
- The applicant remains authenticated.
- The applicant is shown the "Application Rejected" screen with the rejection reason.
- There is no button or link to re-upload evidence.
- The applicant is trapped on this screen.

## 3. Environment Tested
- **Files inspected:** `src/app/(authenticated)/layout.tsx`, `src/app/(authenticated)/onboarding/page.tsx`, `src/app/api/onboarding/submit/route.ts`, `src/app/(authenticated)/reviewer/queue/page.tsx`.
- **Context:** Post Observation 1 implementation.

## 4. Current Rejected-State Route Behavior
The rejected state is enforced by a route guard in the authenticated shell (`src/app/(authenticated)/layout.tsx`):
```tsx
  if (identity && identity.verification_status === 'REJECTED') {
    // ... fetches evidence ...
    return (
      <div className="min-h-screen bg-gray-50 flex flex-col">
         {/* Rejection UI */}
      </div>
    );
  }
```
Because this layout component explicitly returns the rejection UI instead of `{children}` when the status is `REJECTED`, the applicant is completely blocked from rendering **any** nested route within the `(authenticated)` group. The `/onboarding` page is never even reached.

## 5. Existing Onboarding/Evidence-Upload Surface
The existing onboarding page (`src/app/(authenticated)/onboarding/page.tsx`) contains its own explicit gate:
```tsx
  if (identity.verification_status !== 'PENDING') {
    redirect('/');
  }
```
Even if the layout guard were removed, a `REJECTED` user attempting to access the evidence-upload form would be immediately redirected away to `/`. The page explicitly requires `PENDING` status.

## 6. Evidence Upload and Submission Flow
The upload submission endpoint (`POST /api/onboarding/submit/route.ts`) checks the status of the identity:
```tsx
    if (identity.verification_status !== 'PENDING') {
      return NextResponse.json({ error: 'Account is not in pending status' }, { status: 400 });
    }
```
If an applicant somehow bypassed the routing guards, the API itself would reject the submission with a 400 Bad Request. 

When a submission is successful (for `PENDING` users), the API deletes the old `onboarding_evidence` record, increments the version, and inserts a new one. Crucially, the API **does not update** the `freight_identities` table.

## 7. Rejected-Applicant Resubmission Behavior
Currently, resubmission is completely unsupported across all application layers (Layout, Page, API). A rejected applicant cannot reach the form, cannot submit the form, and cannot transition their state.

## 8. Database / State Transition Evidence
Because `/api/onboarding/submit/route.ts` only modifies the `onboarding_evidence` table, it lacks the logic to change `freight_identities.verification_status` from `REJECTED` to `PENDING`.
State Transition: **REJECTED → blocked/error**.

## 9. Reviewer Queue Re-entry Evidence
The Reviewer Queue (`src/app/(authenticated)/reviewer/queue/page.tsx`) explicitly queries:
```tsx
    .from('freight_identities')
    .eq('verification_status', 'PENDING')
```
If a rejected user somehow submitted new evidence without changing their identity status to `PENDING`, they would **not** reappear in the Reviewer queue. They would remain permanently rejected.

## 10. Authentication / Authorization Assessment
The issue is unrelated to the Supabase authentication session. The user is fully authenticated and identity resolution succeeds. The trap is purely a consequence of the application's routing logic and business rules strictly enforcing the `PENDING` state for onboarding.

## 11. Evidence Classification
- The layout route block: **VERIFIED**
- The onboarding page redirection: **VERIFIED**
- The API submission 400 error: **VERIFIED**
- The lack of state transition logic in API: **VERIFIED**
- The reviewer queue PENDING requirement: **VERIFIED**

## 12. Root Cause
The root cause is a **missing state transition architecture for rejected applicants**. The system was designed as a one-way pipeline (`PENDING → VERIFIED | REJECTED`). It lacks the frontend routing, page-level authorization, API submission logic, and database state transitions required to cycle a user from `REJECTED` back to `PENDING`. 

## 13. Contributing Factors
1. **Layout Route Guard:** `layout.tsx` hard-blocks the entire authenticated tree for rejected users.
2. **Page-Level Redirection:** `/onboarding` redirects non-pending users.
3. **API Hard-Stop:** `/api/onboarding/submit` rejects non-pending submissions.
4. **Missing Transition Logic:** The API does not reset `verification_status` to `PENDING`.

## 14. Ruled-out Hypotheses
- **UI Omission:** It is not simply a missing "Re-upload" button. Adding a button to redirect to `/onboarding` would fail because the route is blocked by the layout and the page itself.
- **Authentication Issue:** Supabase auth is working correctly; the user is valid and recognized.

## 15. Recovery Feasibility Assessment
The existing system **does not** currently support recovery.
- Navigation back to onboarding: **Not supported.**
- Evidence replacement: **Not supported for REJECTED users.**
- Resubmission transition: **Not supported.**
- Transition back to review eligibility: **Not supported.**

## 16. Recommended Fix Scope (Recommendation Only)
A complete fix requires coordinated changes across the stack:
1. **Frontend Layout:** Update `layout.tsx` to display a "Re-submit Evidence" action that allows the user to transition to the `/onboarding` page (likely requiring a UI state change or specific query param, or letting them through to the onboarding page while in a REJECTED state).
2. **Frontend Page:** Update `/onboarding` to allow `REJECTED` users to view the form.
3. **API Logic:** Update `/api/onboarding/submit` to accept submissions from `REJECTED` users.
4. **State Transition:** Update the API to execute an explicit database update on `freight_identities`, changing `verification_status` from `REJECTED` back to `PENDING` and clearing the `reviewed_at` timestamp.

## 17. Stop Conditions / Unresolved Questions
None. The root cause is fully identified.

## 18. Final Investigation Status
Investigation complete. Root cause verified. No code changes have been made. Awaiting governance decision on implementation.
