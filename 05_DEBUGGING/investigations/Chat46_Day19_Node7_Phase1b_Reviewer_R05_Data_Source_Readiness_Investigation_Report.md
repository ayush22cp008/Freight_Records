# R-05 Data-Source Readiness Investigation Report
**Investigation:** Chat46 / Day19 / Node 7 / Phase 1b
**Date:** 2026-09-09
**Owner:** Antigravity

---

## 1. Investigation Metadata
- **Objective:** Determine if the existing application structure, data sources, and read mechanisms are sufficient to support the locked Reviewer Portal Verification History blueprint without backend/schema changes.
- **Result:** **R-05 NOT READY**

## 2. Scope and Governing Records Inspected
- `03_IMPLEMENTATION/prompts/Chat46_Day19_Node7_Phase1b_Reviewer_R05_Data_Source_Readiness_Investigation_Handoff.md`
- Codebase paths inspected: `src/app/(authenticated)/reviewer/queue/page.tsx`, `ReviewAction.tsx`, `src/app/api/admin/review/route.ts`
- Migrations inspected: `004_create_freight_identities.sql`, `005_v2_onboarding_evidence.sql`

## 3. Existing Reviewer Frontend Structure Findings
- **Routes:** Only `/reviewer/queue` exists (`page.tsx`). There is no history or detail route.
- **Structure:** `ReviewerQueuePage` fetches pending records directly within the server component using `supabaseServer` (bypassing RLS).
- **UI:** No existing History UI or component capable of rendering completed records. 

## 4. Existing Persistence / Data-Source Findings
- **Storage of Decisions:** Decisions update `verification_status` in `freight_identities` (`VERIFIED` or `REJECTED`) and `status` in `onboarding_evidence`.
- **Fields Available:** `email`, `requested_role` are in `freight_identities`. `rejection_reason`, `document_type`, `storage_path` are in `onboarding_evidence`.
- **Decision Date:** **GAP**. There is no `reviewed_at` or `decision_date` field. The `api/admin/review/route.ts` handler does not update the `updated_at` timestamp. The decision timestamp is lost upon approval/rejection.

## 5. Existing Reviewer-Readable Read/Query Mechanism
- **Mechanism:** **GAP**. No API or server component exists for fetching completed (`VERIFIED`/`REJECTED`) records.
- **Authorization:** `onboarding_evidence` has an RLS policy for Reviewers (`"Reviewers can view all evidence"`). However, `freight_identities` **lacks** any RLS policy for Reviewers (users can only view their own identity). Accessing completed identities would require either a new API endpoint (using `supabaseServer`) or a schema RLS update.

## 6. Evidence Mapping

| Locked requirement | Existing source/mechanism | Evidence | Readiness |
|---|---|---|---|
| Applicant email | `freight_identities.email` | `004_create_freight_identities.sql` | READY (Requires Service Role/New API) |
| Claimed role | `freight_identities.requested_role` | `004_create_freight_identities.sql` | READY |
| Final decision | `freight_identities.verification_status` | `api/admin/review/route.ts` | READY |
| Rejection reason when applicable | `onboarding_evidence.rejection_reason` | `005_v2_onboarding_evidence.sql` | READY |
| Decision date/time | NONE | `api/admin/review/route.ts` | **GAP** |
| Submitted evidence reference | `onboarding_evidence.storage_path` | `005_v2_onboarding_evidence.sql` | READY |
| Completed-record read access | NONE | `src/app/(authenticated)/reviewer` | **GAP** |
| Reviewer authorization | RLS missing on `freight_identities` | `004_create_freight_identities.sql` | **GAP** |
| Chronological History ordering | Cannot order by decision date | N/A | **GAP** |
| Pagination for larger History sets | Not implemented | N/A | **GAP** |

## 7. Authorization / Access Findings
- Reviewers have explicit RLS to view `onboarding_evidence`.
- Reviewers **do not** have RLS to view `freight_identities`, which holds the email and requested role. 

## 8. Evidence-Viewer / Data-Reference Readiness Findings
- Storage paths and signed URL generation logic exists in `ReviewAction.tsx`, which is capable of serving as the foundation for the evidence viewer if adapted for read-only history.

## 9. Boundary / Dependency Findings
Addressing these gaps requires crossing the Phase 1b boundary:
1. **Database/Schema:** Adding a `decision_date` or `reviewed_at` timestamp.
2. **Authorization/RLS:** Adding RLS policies for reviewers on `freight_identities`.
3. **New Backend API:** Adding an API route to fetch completed records if RLS is not used.

## 10. Classification
- **VERIFIED:** Missing decision timestamp, missing RLS on `freight_identities`, missing frontend history mechanism.
- **INFERRED:** None.
- **UNKNOWN:** None.

## 11. Root Cause for Readiness Gap
The initial onboarding implementation focused strictly on the Queue (Pending state). Completed states were ignored, so no timestamps or reviewer-accessible query policies/endpoints were created for historical data.

## 12. Final R-05 Readiness Decision
**R-05 NOT READY**

## 13. Explicit Next-Step Recommendation
Do not begin Reviewer implementation. Escalate the dependency to project-control to authorize backend schema updates (adding `reviewed_at` timestamps) and RLS/API updates for fetching completed records.
