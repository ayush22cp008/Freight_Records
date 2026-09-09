# Reviewer Whole Existing System Investigation Report
**Investigation:** Chat46 / Day19 / Node 7 / Phase 1b (v2)
**Date:** 2026-09-09
**Owner:** Antigravity

---

## 1. Investigation Metadata
- **Objective:** Discover the whole existing Reviewer Portal/system before implementation and compare it against the locked Reviewer Blueprint to determine exact gaps and boundaries.
- **Result:** **R-05 NOT READY**

## 2. Objective and Scope
Investigate the **whole existing Reviewer Portal/system**, establish what exists today, how it works, and assess R-05 readiness (Verification History).

## 3. Governing Records Inspected
- `03_IMPLEMENTATION/prompts/Chat46_Day19_Node7_Phase1b_Reviewer_Whole_Existing_System_Investigation_Handoff_v2.md`
- `02_ARCHITECTURE/locked_blueprints/Reviewer_Locked_Blueprint.md`
- Codebase paths: `src/app/(authenticated)/reviewer/queue/page.tsx`, `ReviewAction.tsx`, `src/app/api/admin/review/route.ts`
- DB schema: `004_create_freight_identities.sql`, `005_v2_onboarding_evidence.sql`

---

## 4. Complete Existing Reviewer Route/Navigation Discovery
- **Routes:** Only one Reviewer route exists: `src/app/(authenticated)/reviewer/queue/page.tsx`.
- **Reviewer entry point:** The Queue page itself.
- **Queue/list routes:** Exists at `/reviewer/queue`.
- **Applicant/detail/review routes:** Do not exist. Review is done inline within the Queue list.
- **Result/success routes:** Do not exist. Page simply refreshes upon decision.
- **History/completed-record routes:** Do not exist.
- **Navigation:** N/A, as it's a single screen.
- **Route protection:** Validated directly in `page.tsx` server component by querying `reviewer_authorizations`.

## 5. Complete Existing Reviewer UI/Component Discovery
- **Queue/list UI:** Hardcoded map over `pendingList` in `page.tsx`.
- **Applicant information display:** Basic inline text showing Email, Requested Role, Evidence Type, and Mime Type.
- **Evidence display/viewer:** Controlled by `ReviewAction.tsx`, generating a signed URL valid for 60s and providing an "Open Document" anchor tag (opens in new tab).
- **Review action & verification controls:** Two buttons ("Approve", "Reject") inside `ReviewAction.tsx`.
- **Rejection-reason UI:** A native browser `prompt()` popup when rejecting.
- **Processing/error states:** Basic boolean state (`disabled={loading}`, `Processing...` text) and inline error text mapping.
- **Result UI:** Handled implicitly via `router.refresh()` removing the item from the pending list.
- **History / completed-record UI:** MISSING.

## 6. Existing Reviewer Workflow Walkthrough
- **Reviewer Entry:** EXISTING
- **Queue:** EXISTING
- **Applicant Review:** EXISTING (Inline)
- **Evidence Examination:** EXISTING (New tab via Signed URL)
- **Existing Verification / Decision Action:** EXISTING
- **Approve / Reject:** EXISTING
- **Existing Result / Post-decision behavior:** EXISTING (Item disappears on refresh)
- **Existing History / Completed Record:** **MISSING**

## 7. Existing State / Persistence Behavior
- **Pending state:** `verification_status = 'PENDING'` (freight_identities) and `status = 'PENDING'` (onboarding_evidence).
- **Verification state:** `verification_status = 'VERIFIED'`, `status = 'APPROVED'`.
- **Rejection state:** `verification_status = 'REJECTED'`, `status = 'REJECTED'`.
- **Opening a record / viewing evidence:** Does not change any persistence state (no "Under Review" lock).
- **Decision commits:** Both tables are updated in a single API call (`/api/admin/review`). Business records (`companies` / `drivers`) are also created.
- **Failure behavior:** HTTP 500 error, UI resets loading state.
- **Completed decision immutability:** Practically immutable in current system as there is no UI or API to reverse a decision.

## 8. Existing Evidence System
- **Storage:** `onboarding_evidence` Supabase bucket.
- **Type tracking:** `document_type`, `mime_type`.
- **Access mechanism:** `ReviewAction.tsx` calls `supabase.storage.createSignedUrl(path, 60)`.
- **Error handling:** Sets a generic frontend error if the URL fails to generate.
- **Relationship:** Linked via `auth_id` to the applicant's identity.

## 9. Existing Decision Mechanism
- **Frontend caller:** `handleAction` inside `ReviewAction.tsx`.
- **API route:** POST `/api/admin/review`.
- **Persistence:** Updates `status` fields, saves `rejection_reason`.
- **Timestamps / Metadata:** **MISSING**. Only `created_at` and `updated_at` exist, but `updated_at` is not updated during the API call. The time of the decision is lost.

## 10. Existing Data Model
- **Applicant identifier/email:** `freight_identities.email`.
- **Claimed/requested role:** `freight_identities.requested_role`.
- **Verification status:** `freight_identities.verification_status`.
- **Evidence reference:** `onboarding_evidence.storage_path`.
- **Rejection reason:** `onboarding_evidence.rejection_reason`.
- **Decision metadata/Timestamps:** **MISSING**.
- **Completed-record info:** **MISSING**.

## 11. Existing Read/Query Mechanisms
- **Route:** `page.tsx` server component.
- **Source table:** `freight_identities` and `onboarding_evidence`.
- **Fields returned:** `*`.
- **Authorization:** `supabaseServer` service role is used. This bypasses normal RLS.
- **Supports completed records:** No, hardcoded to filter by `PENDING`.

## 12. Existing Authorization/Security Boundaries
- Reviewers **do not** have an RLS policy to read `freight_identities` (users can only read their own).
- Reviewers **do** have an RLS policy to read `onboarding_evidence`.
- Current read mechanism bypasses RLS by using the admin `supabaseServer` client for the Queue page.

---

## 13. Existing System vs Locked Blueprint Comparison

| Locked capability | Existing implementation | Evidence/path | Status |
|---|---|---|---|
| Reviewer entry | `/reviewer/queue` route | `queue/page.tsx` | EXISTING |
| Verification Queue | Fetches `PENDING` records | `queue/page.tsx` | EXISTING |
| Applicant Verification | Inline inside Queue | `ReviewAction.tsx` | EXISTING (Partial UI) |
| Applicant email context | `identity.email` | `queue/page.tsx` | EXISTING |
| Claimed Role context | `identity.requested_role` | `queue/page.tsx` | EXISTING |
| Submitted Evidence access | `storage_path` reference | `ReviewAction.tsx` | EXISTING |
| Evidence Examination | Opens signed URL in new tab | `ReviewAction.tsx` | EXISTING |
| Identity / Role Verified action | `/api/admin/review` | `ReviewAction.tsx` | EXISTING |
| Approve | Admin Review API | `api/admin/review/route.ts` | EXISTING |
| Reject | Admin Review API | `api/admin/review/route.ts` | EXISTING |
| Required rejection reason | Browser `prompt()` | `ReviewAction.tsx` | EXISTING (Partial UI) |
| Processing state | Simple `loading` boolean | `ReviewAction.tsx` | EXISTING |
| Decision failure handling | `try/catch` and alert | `ReviewAction.tsx` | EXISTING |
| Decision Result | Refreshes queue | `queue/page.tsx` | EXISTING |
| Verification History | Not implemented | N/A | **MISSING** |
| Chronological ordering | Missing timestamp data | N/A | **MISSING** |
| Pagination | Not implemented | N/A | **MISSING** |
| Read-only completed record | Not implemented | N/A | **MISSING** |
| Completed-record evidence viewer | Not implemented | N/A | **MISSING** |
| History navigation | Not implemented | N/A | **MISSING** |
| Completed decision immutability | No reversal capability exists | N/A | EXISTING |

---

## 14. R-05 Readiness Assessment
Can the existing system provide the data, access, and read/query foundation required for the locked Verification History experience?
**NO.** The system lacks the critical timestamps to order decisions, lacks RLS policies for reviewers on identity data, and lacks the frontend queries to fetch completed records.

## 15. Classification
- **VERIFIED:** Missing decision timestamp in schema/API; missing Reviewer RLS on `freight_identities`; missing frontend history mechanism.
- **INFERRED:** None.
- **UNKNOWN:** None.

## 16. Root Causes for Important Gaps
The existing implementation was built as an MVP for processing Pending requests only. As a result, historical metadata (like `reviewed_at`) was omitted, and security policies were kept strict since server-side bypassing (via `supabaseServer`) was sufficient for the single Queue page.

## 17. Boundary/Dependency Analysis
1. **Database/Schema:** Must add a `decision_date` or `reviewed_at` timestamp.
2. **Authorization/RLS:** Must update RLS policies for Reviewers on `freight_identities`, or create a new Admin API endpoint for History.
3. Both of these changes cross the Phase 1b frontend-only redesign boundary and require explicit backend/schema project-control authorization.

## 18. Final R-05 Decision
**R-05 NOT READY**

## 19. Recommended Governance Next Step
Do not begin Reviewer UI implementation. Escalate the boundary-crossing dependencies (Schema update for timestamps, API/RLS update for completed records access) through project-control to authorize these backend changes before frontend implementation begins.
