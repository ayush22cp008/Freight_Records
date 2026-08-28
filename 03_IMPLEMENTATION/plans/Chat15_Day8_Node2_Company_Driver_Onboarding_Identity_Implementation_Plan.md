# Node 2 Company & Driver Onboarding Identity Implementation Plan

## Goal Description
Implement the complete onboarding, evidence collection, and verification flow for Drivers and Companies. This involves updating signup to capture `requested_role`, collecting role-specific evidence (Driving Licence or GST), providing a secure reviewer workflow to approve identities and assign `trusted_role`, and implementing strict role-based routing in the authenticated application area.

## User Review Required
> [!IMPORTANT]
> - **Companies Table & Generated Driver Codes:** Approving a Driver will automatically generate a simple `driver_code` in the `drivers` table for MVP purposes. A new `companies` table will be created to house verified Company profiles.
> - **Evidence Storage:** For the MVP, evidence (GST or Driving Licence) will be collected as simple text values stored in an `onboarding_evidence` table rather than full file uploads.

## Open Questions
- Is text-based evidence acceptable for the MVP, or do we strictly need file uploads right now? (Assuming text-based is fine for this Node).
- Is it acceptable for the Reviewer Workflow to be a set of secure API endpoints that you can call manually (or via a simple hidden admin UI), rather than a full-fledged admin dashboard?

## Proposed Changes

---

### Database Foundation
#### [NEW] `src/db/migrations/005_create_onboarding_evidence.sql`
- Creates `companies` table (`id`, `auth_id`, `name`).
- Creates `onboarding_evidence` table (`auth_id`, `role_type`, `document_type`, `document_value`, `status`).
- Enables RLS on both tables (users can view/insert their own evidence; service role handles companies).

---

### Core Application
#### [MODIFY] `src/app/signup/page.tsx`
- Adds a radio button selector for `requested_role` (Driver vs. Company).
- Passes the selected role in the `supabase.auth.signUp` metadata.

#### [NEW] `src/app/(authenticated)/onboarding/page.tsx`
- A dedicated onboarding form for users with a `PENDING` identity.
- Dynamically asks for "Driving Licence" or "GST Details" based on `requested_role`.
- Once submitted, it displays a "Verification Pending" state.

#### [NEW] `src/app/api/onboarding/submit/route.ts`
- Secure API route to accept evidence and insert it into `onboarding_evidence`.

#### [NEW] `src/app/api/admin/review/route.ts`
- A protected (service-role) API endpoint for you to approve/reject onboarding requests.
- **On Approval:** Updates `freight_identities` (`verification_status = 'VERIFIED'`, `trusted_role = requested_role`). If Driver, creates a `drivers` record; if Company, creates a `companies` record.

#### [MODIFY] `src/app/(authenticated)/layout.tsx`
- Enforces routing:
  - `VERIFIED`: Renders main app.
  - `PENDING`: Restricts access to ONLY the onboarding flow (`/onboarding`).
  - `REJECTED`: Renders a rejected boundary.

#### [MODIFY] `src/app/(authenticated)/page.tsx`
- Modifies the dashboard to branch based on `trusted_role`.
- Shows the existing Driver UI for `DRIVER`s.
- Shows a new Company placeholder dashboard for `COMPANY`s.

## Verification Plan

### Automated Tests
- Build and Type check (`npm run build`, `npx tsc --noEmit`).

### Manual Verification Checklist (for Ayush)
1. **Driver Signup:** Sign up, select Driver. Submit "Licence" in onboarding. Wait at "Pending Review".
2. **Company Signup:** Sign up, select Company. Submit "GST" in onboarding. Wait at "Pending Review".
3. **Reviewer Approval:** Call the admin API (or use the provided admin snippet) to approve both users.
4. **Driver Routing:** Log in as the verified Driver -> See Driver Dashboard.
5. **Company Routing:** Log in as the verified Company -> See Company Dashboard.
6. **Authorization Tampering:** Attempt to modify roles via browser dev tools and confirm access is denied.
