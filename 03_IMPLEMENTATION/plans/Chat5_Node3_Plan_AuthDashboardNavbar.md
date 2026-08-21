# Chat5 Node3 — Implementation Plan: Authentication + Dashboard + Navbar

**Status:** PROPOSED — REVIEW BEFORE EXECUTION
**Scope:** Basic authentication + authenticated application shell + Dashboard/Navbar
**Basis:** Authentication investigation + Auth/Dashboard UI investigation

## 1. Objective

Build a coherent authenticated application foundation before continuing the freight event workflow.

The goal is to avoid disconnected pages and future rewiring.

Final high-level flow:

```text
Create Account
      ↓
Login
      ↓
Authenticated App Shell
      ↓
Dashboard
      ↓
Active Trip workflow
      ↓
Arrival → Check-in → Departure → Timeline
```

## 2. Authentication

Replace the temporary driver-code-only login with basic account authentication using the safest supported Supabase Auth approach.

Required MVP capabilities:

- Create Account
- Email + Password
- Login
- Persistent authenticated session
- Sign out
- Already-authenticated user visiting `/login` → redirect to `/`
- Unauthenticated user visiting protected pages → redirect to `/login`

Do NOT add password reset, email verification UX, profile settings, roles/admin, or other non-essential auth features unless the existing Supabase setup requires them for correctness.

## 3. Driver identity mapping

Preserve the existing freight data model.

Authenticated Supabase user identity must map cleanly to the existing `drivers` record and therefore to the driver's active trip.

Do NOT create a new trip architecture.
Do NOT redesign `trips` or `events` merely to support authentication.

The exact mapping strategy must be confirmed from the existing source/database before implementation.

If a schema change is strictly required, stop and document it before applying it rather than silently changing the schema.

## 4. Authenticated application shell

Create a shared authenticated layout/application shell.

Preferred structure, subject to source compatibility:

```text
src/app/(authenticated)/layout.tsx
```

The shell should contain the shared Navbar and render authenticated pages beneath it.

`/login` and account creation should remain outside the authenticated shell.

Do not duplicate Navbar markup across pages.

## 5. Navbar

Create one reusable Navbar component.

MVP navigation:

- Dashboard
- Active Trip
- Timeline
- Sign out

Important:

- Dashboard and Navbar appear together on the same screen when the driver is on Dashboard.
- Navbar is a shared application-shell component, not Dashboard-specific markup.
- Navbar should also appear on authenticated workflow pages.
- Do not add unnecessary Profile, Settings, Admin, or other navigation items.
- Sign out must actually terminate the authenticated session and return to `/login`.
- The Navbar should display the authenticated driver's identity only if that information is already safely available without unnecessary queries.

### Active Trip vs Dashboard

Do not duplicate the active-trip workflow unnecessarily.

The Dashboard is the primary workflow hub and should show the active trip and next required action.

If a separate Active Trip route is implemented, it must have a clear purpose beyond duplicating the Dashboard. If the current source does not justify a separate page, keep Dashboard as the active-trip hub and do not create a redundant page.

## 6. Dashboard

Transform the current `/` Hub into the authenticated Dashboard.

Minimum content:

- Active Trip
- Facility name
- Current event progress
- Completed events
- Next required event
- One clear primary CTA

Expected state logic remains authoritative and database-driven:

```text
No Arrival
→ Start Arrival

Arrival complete
→ Start Check-in

Check-in complete
→ Start Departure

Departure complete
→ View Timeline
```

Do not replace the existing state logic with client-only state.

## 7. Existing event workflow integration

Preserve the current Arrival implementation and connect it cleanly to the new authenticated shell.

Do NOT implement the complete Check-in, Departure, Timeline, or AI Summary in this task unless required only to prevent a structural routing failure.

The later workflow remains:

```text
Dashboard
 ↓
Arrival
 ↓
Dashboard
 ↓
Check-in
 ↓
Dashboard
 ↓
Departure
 ↓
Timeline
 ↓
AI Summary
```

Temporary missing-page behavior is acceptable for pages that are intentionally deferred, but the authenticated shell architecture must make their later integration straightforward.

## 8. Route protection

Implement consistent route protection.

Required:

- `/login` accessible while unauthenticated
- account creation accessible while unauthenticated
- authenticated user visiting `/login` → `/`
- protected Dashboard/workflow routes require authenticated session
- direct URL access cannot bypass authentication
- refresh preserves valid session
- sign out removes access to protected pages

Do not rely only on client-side redirects for security-critical protection.

## 9. Session handling

Use the official Supabase Auth session mechanism appropriate for the current Next.js/Supabase setup.

Avoid retaining the old custom `driver_id` cookie as a second independent authentication authority unless there is a documented transitional reason.

The authenticated Supabase user should become the source of authentication truth.

Any transition compatibility required by the current code must be explicit and temporary.

## 10. UI design

Use the existing Tailwind approach.

Do not introduce a large component library or unnecessary design system.

Create reusable components only where reuse is real, especially:

- Navbar
- authenticated shell/layout
- common navigation elements

Keep the existing visual style unless a small adjustment is needed for consistency.

The Dashboard should feel like the central application page, not another isolated event page.

## 11. Account creation

Create a simple account page/form with:

- Email
- Password
- Confirm Password if needed for basic UX validation
- Create Account action
- Link to Login

Handle basic validation and clear errors.

Do not add advanced account management features.

## 12. Login

Update the existing Login UI to:

- Email
- Password
- Login
- Link to Create Account
- Clear authentication errors
- Loading state
- Redirect authenticated users appropriately

Preserve the existing visual style where practical.

## 13. Sign out

Add Sign out to the Navbar.

Expected behavior:

```text
Authenticated user
 ↓
Sign out
 ↓
Supabase session terminated
 ↓
/login
```

After sign out, direct access to `/` must not remain available.

## 14. Database and security constraints

- Preserve RLS/security model.
- Do not weaken policies to make authentication easier.
- Do not store plaintext passwords in the application database.
- Prefer Supabase Auth for password storage/credential handling.
- Do not expose service-role credentials to the browser.
- Preserve immutable event behavior.
- Do not modify event records as part of this task.

## 15. Implementation order

Recommended order:

### Phase A — Auth foundation
1. Confirm Supabase Auth configuration.
2. Implement account creation.
3. Implement email/password login.
4. Establish server-aware persistent session.
5. Implement sign out.
6. Implement authenticated/unauthenticated route behavior.

### Phase B — Application shell
7. Create authenticated layout.
8. Create reusable Navbar.
9. Place Dashboard inside authenticated shell.
10. Integrate existing Arrival route with shell.

### Phase C — Dashboard
11. Preserve/clean current DB-driven next-event logic.
12. Display active trip/progress/next action.
13. Add Navbar navigation.
14. Ensure refresh/direct-route behavior is correct.

### Phase D — Verification
15. Build/test.
16. Test create account.
17. Test login.
18. Test refresh/session persistence.
19. Test authenticated `/login` redirect.
20. Test sign out.
21. Test protected route after sign out.
22. Test Dashboard → Arrival.
23. Test existing Arrival completion → Dashboard.
24. Confirm no duplicate Arrival behavior regresses.

## 16. Do NOT implement

- Multi-stop trips
- Pickup/delivery-specific event taxonomy
- In-transit event
- Admin dashboard
- Driver management UI
- Password reset
- Advanced profile/settings
- Roles/permissions unless already required by the existing source
- Full Check-in implementation
- Full Departure implementation
- Timeline implementation
- AI summary implementation
- New database architecture

## 17. Completion criteria

The task is complete when:

- A user can create an account with email/password.
- The user can log in.
- A valid session persists through refresh.
- An authenticated user cannot remain on `/login` as a normal login screen; they are redirected to Dashboard.
- Dashboard is protected.
- Navbar appears on authenticated pages.
- Navbar contains only the agreed MVP navigation.
- Sign out terminates the session and returns to Login.
- Dashboard remains the workflow source of truth.
- Existing Arrival behavior still works.
- No event/database security regressions are introduced.
- Build/tests pass or failures are explicitly documented.

## 18. Required implementation report

After implementation, write:

`03_IMPLEMENTATION/implementation_reports/Chat5_Node3_Report_AuthDashboardNavbar.md`

Report:

1. Files changed
2. Authentication implementation
3. Session behavior
4. Driver identity mapping
5. Authenticated layout
6. Navbar
7. Dashboard
8. Route protection
9. Sign-out behavior
10. Existing Arrival integration
11. Database/schema changes, if any
12. Security considerations
13. Build/test results
14. Manual verification steps
15. Known limitations

Do not claim manual browser tests were completed unless actually performed.

## Final execution rule

This plan must be reviewed and explicitly approved before Antigravity modifies the source repository.
