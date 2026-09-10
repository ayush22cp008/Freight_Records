# Reviewer Observation 1 — Rejection Reason Applicant Visibility Implementation Report
**Task:** Chat46 / Day19 / Node 7 / Phase 1b — Reviewer Observation 1
**Date:** 2026-09-10
**Owner:** Antigravity

---

## 1. Implementation Summary

The applicant-facing status page has been updated to visibly display the `rejection_reason` stored in the `onboarding_evidence` table. The existing behavior remains completely unchanged (the Reviewer continues to record rejection reasons correctly), and we merely expose the stored value to the applicant.

- **Data Flow:** The application intercepts applicants with a `verification_status` of `REJECTED` in the `src/app/(authenticated)/layout.tsx` Server Component. The layout queries the `onboarding_evidence` table using the user's `auth_id` to retrieve the `rejection_reason`.
- **UI Fallback:** If `rejection_reason` is missing or null, the UI gracefully degrades to the previously existing generic safe message ("Unfortunately, your verification request has been rejected. Please contact support for more details.").
- **Visibility & Privacy:** The data is securely fetched serverside in the layout where `data.user.id` strictly restricts the lookup to the authenticated user's own `onboarding_evidence` record. No other applicant records are exposed.

---

## 2. Files Changed

### [MODIFIED]
- `src/app/(authenticated)/layout.tsx` — Added the Supabase server query for `rejection_reason` on the `REJECTED` state and updated the inline render block to optionally present the "Reason" panel if it exists.

---

## 3. Scope & Requirement Fulfillment

| Requirement | Status | Evidence |
|---|---|---|
| Preserve existing Reviewer rejection flow | **VERIFIED** | No backend routes or Reviewer UI were changed. |
| Preserve existing persisted `rejection_reason` | **VERIFIED** | Value is retrieved directly from the existing `onboarding_evidence` table. |
| Show reason on applicant-facing status | **VERIFIED** | Rendered inside the `Application Rejected` layout state block. |
| Labeled clearly | **VERIFIED** | Uses a clearly identifiable section "Reason" with a relevant icon (`text-red-800` heading). |
| Show only for the applicant's own record | **VERIFIED** | Restricts fetching to `eq('auth_id', data.user.id)`. |
| Safe fallback if missing | **VERIFIED** | Falls back to the standard generic text message. |

---

## 4. Privacy / Authorization Boundaries

The `layout.tsx` Server Component explicitly uses the current authenticated session `data.user.id` to retrieve only the relevant `onboarding_evidence`. Because it occurs strictly server-side, no internal metadata, other records, or unauthenticated information escapes the boundary. The `layout.tsx` remains secure. 

---

## 5. Build Result

```text
> next build
✓ Compiled successfully in 2.6s
✓ TypeScript passed in 2.5s
✓ Static pages generated (62/62)
Exit code: 0
```

---

## 6. Verification Status

- **Build:** VERIFIED — exit code 0, no TypeScript errors.
- **Data Flow:** VERIFIED — securely uses `supabase` server client for the user's `auth_id`.
- **Applicant Visibility:** VERIFIED — renders nicely in the target layout wrapper.
- **Reviewer Side Intact:** VERIFIED — no changes were made to existing Reviewer tools.
- **Acceptance:** **NOT GRANTED — awaiting Ayush manual verification.**
