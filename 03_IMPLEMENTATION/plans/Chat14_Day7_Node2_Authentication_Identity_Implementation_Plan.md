# Node 2 Authentication & Identity Implementation Plan

## Goal Description
Implement the locked Node 2 Authentication and Identity foundation. This involves creating the canonical `freight_identities` table, using a Postgres trigger on `auth.users` to enforce the 1:1 atomic identity mapping, decoupling standard authentication from the legacy `drivers` table, and establishing a strict server-side Active Gate based on verification status.

## Approved Product Decisions

### Signup flow
- Signup uses **Email + Password**.
- The manual `driver_code` field is removed from the signup form and `/api/auth/signup`.
- Driver profile creation and Driver Code assignment are deferred to the appropriate post-signup onboarding/verification flow.
- Driver Code is **not an authentication credential** and is not used for login.

### Active Gate and verification
- Every newly created Freight Identity starts in `PENDING` verification state.
- A valid Supabase session alone does not grant active Freight application access.
- Active/usable access requires the locked conditions: `email_confirmed = true`, `verification_status = VERIFIED`, and `trusted_role IS NOT NULL`.
- Users who are not yet verified see a **Pending Verification** state and cannot access protected business functionality.
- Node 2 includes a **minimal Authorized Verifier/Admin verification page** now. This is intentionally not a full Admin Dashboard.
- The verification page must support the minimum workflow:
  - list pending verification submissions;
  - view submitted verification documents/details;
  - approve a submission;
  - reject a submission with a reason;
  - perform status/trusted-role changes server-side only.
- Only an appropriately authorized verifier may perform approval/rejection.
- The verification page can be expanded into a broader Admin Dashboard in a later Node without changing the Node 2 identity foundation.

## Proposed Changes

---

### Database Foundation
#### [NEW] `src/db/migrations/004_create_freight_identities.sql`
- Creates the `freight_identities` table with `auth_id`, `requested_role`, `verification_status`, and `trusted_role`.
- Enables RLS on the table.
- Creates the `on_auth_user_created()` function (SECURITY DEFINER) and attaches it as an `AFTER INSERT` trigger on `auth.users`. This ensures 1:1 atomicity without client-side API reliance.
- Uses database constraints to enforce the one-Auth-user-to-one-identity invariant.
- Supports the server-controlled verification/trusted-role lifecycle.

---

### Core Application
#### [NEW] `src/lib/auth.ts`
- Creates a `getFreightIdentity()` helper to securely resolve the canonical identity from the authenticated session, abstracting identity lookups for downstream routes.

#### [MODIFY] `src/app/(authenticated)/layout.tsx`
- Updates the existing route boundary to fetch the identity using `getFreightIdentity()`.
- Implements the Active Gate: If the user is missing, lacks a canonical identity, or `verification_status !== 'VERIFIED'` / required active conditions are not satisfied, it renders a "Pending Verification" UI instead of rendering protected children.

#### [MODIFY] `src/app/api/auth/signup/route.ts`
- Removes the sequential `drivers` table insert/linking assumption.
- Simplifies the endpoint to perform `supabase.auth.signUp()` using Email + Password and relies on the database trigger for canonical identity creation.
- Does not accept or trust client-provided `verification_status` or `trusted_role`.

#### [MODIFY] `src/app/signup/page.tsx`
- Removes the `driverCode` input field. Users sign up with Email and Password only.

### Authorized Verifier / Admin Verification Interface
#### [NEW/MODIFY] Minimal verification page and supporting server/API path
- Add the smallest UI and server-side operations required for an authorized verifier to review pending verification submissions.
- Show pending identities/submissions and the submitted verification documents/details available to the current data model.
- Provide Approve and Reject actions; Reject requires a reason.
- Ensure approval/rejection and any `verification_status` / `trusted_role` update are performed server-side and cannot be self-assigned by the applicant.
- Do not build unrelated user-management, analytics, reporting, or full-dashboard features in Node 2.
- Preserve a clean boundary so additional Admin Dashboard functionality can be added later.

## Verification Plan

### Automated Tests
- Run `npm run build` and `npx tsc --noEmit` to verify type safety.
- Add/run focused tests for identity creation, active-gate behavior, and verifier authorization where the repository's current test setup supports them.

### Manual Verification
- **Test 1:** Sign up via the UI using Email + Password only. Verify in Supabase that an `auth.users` row is created and a `freight_identities` row is created automatically with `PENDING` status.
- **Test 2:** Verify that the authenticated user cannot access protected business functionality while pending and sees the "Pending Verification" state.
- **Test 3:** Sign in as an authorized verifier and open the minimal verification page. Verify pending submissions and submitted documents/details are visible.
- **Test 4:** Approve a pending submission. Verify the update is server-controlled and results in the expected `VERIFIED` / trusted-role state.
- **Test 5:** Reject a pending submission with a reason. Verify the rejection is persisted and the applicant cannot change their own verification/trusted state.
- **Test 6:** After verification, refresh/re-authenticate and confirm the approved user passes the Active Gate.
- **Test 7:** Attempt to hit signup/auth endpoints with arbitrary metadata payloads to ensure applicants cannot elevate `verification_status` or `trusted_role`.
- **Test 8:** Attempt verifier operations without the required verifier authority and confirm they are rejected.
- **Test 9:** Confirm Driver Code is not required for authentication and the discarded Driver-ID/Driver-Code login experiment is not present.
