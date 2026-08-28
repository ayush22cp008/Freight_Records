# Chat 15 Node 2 Implementation Report vs. Locked Plan Investigation

## 1. Executive Summary
This investigation compared the recently completed Chat 15 Node 2 Company & Driver Onboarding Identity implementation against the architectural requirements of the "Locked Plan" (as referenced in the investigation prompt). The investigation confirms that the implemented code and database schema use a simplified "raw-string/headless API" approach for MVP rather than the mandated private Supabase Storage, signed URLs, and dedicated Reviewer Queue. 

## 2. Inspected Artifacts
**Files Inspected:**
- `src/db/migrations/005_create_onboarding_evidence.sql`
- `src/app/signup/page.tsx`
- `src/app/api/auth/signup/route.ts`
- `src/app/(authenticated)/onboarding/page.tsx`
- `src/app/(authenticated)/onboarding/OnboardingForm.tsx`
- `src/app/api/onboarding/submit/route.ts`
- `src/app/api/admin/review/route.ts`
- `src/app/(authenticated)/layout.tsx`
- `src/app/(authenticated)/page.tsx`

**Database Objects Inspected:**
- `onboarding_evidence` table
- `companies` table
- `drivers` table (interaction point)

## 3. Current Behavior vs. Locked Plan Requirements

### A. Database
1. **What did `005_create_onboarding_evidence.sql` create?** 
   A `companies` table (id, auth_id, name, created_at) and an `onboarding_evidence` table (id, auth_id, role_type, document_type, document_value, status, created_at). Both have RLS enabled.
2. **Does `onboarding_evidence` contain a plain-text document field?** 
   **Yes**. It uses `document_value text NOT NULL`.
3. **Does a private Supabase Storage bucket/object path exist?** 
   **No**. Neither bucket creation scripts nor storage policies exist in the migration.
4. **Is there an actual `reviewer_authorizations` table?** 
   **No**. It does not exist.
5. **What tables exist for minimal profiles?** 
   The `drivers` and `companies` tables are populated.
6. **Do the role records use the approved `auth_id` relationship?** 
   **Yes**. Both `drivers.auth_id` and `companies.auth_id` are used.
7. **What RLS policies exist on the new tables?** 
   Users can INSERT and SELECT their own `onboarding_evidence`. Companies can SELECT their own `companies` record.

### B. Source Implementation Conformance

| Requirement | Current Evidence | Status |
|---|---|---|
| Explicit Driver/Company signup selection | `signup/page.tsx` has radio buttons. | VERIFIED |
| `requested_role` capture | Passed to `supabase.auth.signUp()`. | VERIFIED |
| Driver → Licence / Company → GST | `OnboardingForm.tsx` branches correctly. | VERIFIED |
| Actual document upload / Private Storage | Form uses `<input type="text">`. No storage API used. | **CONFLICT** |
| Technical file metadata | Not implemented. | **CONFLICT** |
| `reviewer_authorizations` table | Not implemented. | **CONFLICT** |
| Reviewer Queue + Review page | No UI created. `api/admin/review` is headless. | **CONFLICT** |
| Signed URL evidence access | Not implemented. | **CONFLICT** |
| Transactional approval | Headless API does sequential `await` updates without SQL transactions. | **CONFLICT** |
| Immutable evidence/version history | Not implemented. Table overwrites or appends loosely. | **CONFLICT** |
| Minimal Driver / Company profile | Provisioned via headless API upon APPROVE action. | VERIFIED |
| Role-aware routing | `page.tsx` and `layout.tsx` branch on `trusted_role`. | VERIFIED |

## 4. Root Cause of Stale Implementation
The AI agent formulated the `Chat15_Day8_Node2_Company_Driver_Onboarding_Identity_Implementation_Plan.md` based strictly on the provided implementation prompt, which lacked the explicit architectural requirements for file storage and a reviewer queue. The agent explicitly listed "Open Questions" in the plan asking:
1. *Is text-based evidence acceptable for the MVP, or do we strictly need file uploads right now?*
2. *Is it acceptable for the Reviewer Workflow to be a set of secure API endpoints... rather than a full-fledged admin dashboard?*

Because the user operates an automated prompt-chaining workflow, the plan was blindly approved and pushed to GitHub without answering the questions or supplying the `02_ARCHITECTURE/Chat15_Day8_Node2_Onboarding_Verification_Design_Decision.md` document. The AI therefore assumed the simplified MVP approach was acceptable and executed it.

## 5. Exact Gaps Requiring Correction
- `onboarding_evidence` schema must be replaced to track Supabase Storage object paths, technical metadata (MIME type, size), and version history.
- `OnboardingForm.tsx` and `api/onboarding/submit/route.ts` must be refactored to support actual multipart file uploads to a private Supabase Storage bucket.
- A `reviewer_authorizations` table must be created to securely grant Reviewer permissions independently of `trusted_role`.
- A "Reviewer Queue" UI must be built to allow authorized reviewers to list `PENDING` users, generate signed URLs to view evidence, and click Approve/Reject.
- The approval workflow must be wrapped in a secure, atomic SQL transaction.

## 6. Safety & Migration Assessment
- **Can the existing migration be safely superseded?** 
  Since this is a development environment, `005_create_onboarding_evidence.sql` should be **deleted and replaced** (or `down` migrated) because the current `onboarding_evidence` table structure is fundamentally wrong (relying on `document_value` text). The `companies` table structure is fine but can be bundled into the correct replacement migration.
- **Is a new architecture decision required?** 
  No. The locked architecture decision (`Chat15_Day8_Node2_Onboarding_Verification_Design_Decision.md`) already exists. It just needs to be fed to the implementation agent as part of the execution context.

## 7. Recommended Next Action
**DO NOT ATTEMPT TO PATCH THE CODE IN THIS STEP.**
1. Drop the `onboarding_evidence` and `companies` tables from the local database.
2. Provide the agent with the exact contents of the `Chat15_Day8_Node2_Onboarding_Verification_Design_Decision.md` and the `Chat15_Day8_Node2_Identity_Mapping_OptionB_Decision.md` files.
3. Issue a new prompt to write a **V2 Implementation Plan** that strictly adheres to the Private Storage, Signed URLs, and Reviewer Queue requirements before writing any code.
