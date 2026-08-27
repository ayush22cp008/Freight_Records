# Node 2 Authentication & Identity Implementation Plan

## Goal Description
Implement the locked Node 2 Authentication and Identity foundation. This involves creating the canonical `freight_identities` table, using a Postgres trigger on `auth.users` to enforce the 1:1 atomic identity mapping, decoupling standard authentication from the legacy `drivers` table, and establishing a strict server-side Active Gate based on verification status.

## User Review Required
> [!IMPORTANT]
> - **Signup flow changes:** The manual `driver_code` field will be completely removed from the signup form and the `/api/auth/signup` API route. Driver profile creation (and driver codes) will be deferred to a post-signup onboarding/verification step, ensuring signup only establishes the generic identity.
> - **Active Gate enforcement:** Users who sign up will have a `PENDING` status. The authenticated layout will block access to the main application for users unless their `verification_status` is `VERIFIED`. They will see a "Pending Verification" message instead. (For testing, you will need to manually set `verification_status = 'VERIFIED'` in the database to access the app).

## Open Questions
- Is a simple "Pending Verification" screen acceptable for unverified users in the authenticated layout, or should we redirect them to a specific `/pending` route? (I will use a simple inline UI in the layout for now).

## Proposed Changes

---

### Database Foundation
#### [NEW] `src/db/migrations/004_create_freight_identities.sql`
- Creates the `freight_identities` table with `auth_id`, `requested_role`, `verification_status`, and `trusted_role`.
- Enables RLS on the table.
- Creates the `on_auth_user_created()` function (SECURITY DEFINER) and attaches it as an `AFTER INSERT` trigger on `auth.users`. This ensures 1:1 atomicity without client-side API reliance.

---

### Core Application
#### [NEW] `src/lib/auth.ts`
- Creates a `getFreightIdentity()` helper to securely resolve the canonical identity from the authenticated session, abstracting identity lookups for downstream routes.

#### [MODIFY] `src/app/(authenticated)/layout.tsx`
- Updates the existing route boundary to fetch the identity using `getFreightIdentity()`.
- Implements the Q2 Active Gate: If the user is missing or `verification_status !== 'VERIFIED'`, it renders a "Pending Approval" UI instead of rendering the protected children.

#### [MODIFY] `src/app/api/auth/signup/route.ts`
- Removes the sequential `drivers` table insert.
- Simplifies the endpoint to only perform `supabase.auth.signUp()`, relying strictly on the database trigger for atomic identity creation.

#### [MODIFY] `src/app/signup/page.tsx`
- Removes the `driverCode` input field. Users sign up with only Email and Password.

## Verification Plan

### Automated Tests
- Run `npm run build` and `npx tsc --noEmit` to verify type safety.

### Manual Verification
- **Test 1:** Sign up via the UI. Verify in Supabase that an `auth.users` row is created, AND a `freight_identities` row is created automatically with `PENDING` status.
- **Test 2:** Verify that the user is immediately routed to the app but sees the "Pending Verification" screen, blocking access to business functions.
- **Test 3:** Manually update the `verification_status` to `'VERIFIED'` via the SQL Editor. Refresh the page and confirm the user gains access to the protected application.
- **Test 4:** Attempt to hit the signup API with arbitrary metadata payloads to ensure they cannot elevate `verification_status` or `trusted_role`.
