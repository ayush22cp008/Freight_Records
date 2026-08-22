# Chat7 Node3 Report: Authentication Redesign Investigation

## 1. Executive Summary
This investigation analyzed the current state of authentication in the Freight MVP against the proposed target design (Driver ID + Password login). The current architecture safely isolates the user via Supabase server-side session checks, completely bypassing RLS by using service-role API routes. The primary gap is that the current flow requires an email for login and a predefined `driver_code` for signup, whereas the target design expects the system to *generate* the Driver ID upon signup, email it to the user, and use it in place of an email during login. A server-side proxy approach for resolving Driver ID to email is feasible and secure.

## 2. Current Authentication Architecture
- **Tech Stack**: Next.js 16.3 (App Router), React 19, Supabase SSR (`@supabase/ssr`), Supabase JS Client (`@supabase/supabase-js`).
- **Flow**: The client submits credentials to Next.js API route handlers (`/api/auth/signup`, `/api/auth/login`). These routes use `createClient()` (which reads/writes cookies via `@supabase/ssr`) to interact with Supabase Auth.
- **Session Management**: Supabase automatically sets an HTTP-only session cookie.
- **Ownership Resolution**: Protected API routes (like `/api/events/arrival/route.ts`) verify the cookie via `supabase.auth.getUser()`, and then use the `user.id` to look up the `drivers` record using a service-role client (`supabaseServer`).

## 3. Current `drivers` / `auth.users` Relationship
**Schema Evidence (`src/db/migrations/001_create_core_tables.sql`, `003_add_auth_id_to_drivers.sql`)**:
- `drivers.id` (UUID, Primary Key)
- `drivers.driver_code` (Text, UNIQUE, NOT NULL)
- `drivers.auth_id` (UUID, UNIQUE, Foreign Key referencing `auth.users(id)`)
- *Note:* The `drivers` table does **not** store the driver's email address. Email resides purely in `auth.users`.

**Current Relationship**: `drivers.auth_id` maps 1:1 to `auth.users.id`. `driver_code` is an application-level string (currently expected to be pre-populated in the DB before signup).

## 4. Current Supabase Auth Configuration
**Flow Evidence (`src/app/api/auth/login/route.ts`, `/signup/route.ts`)**:
- **Signup**: Requires `email`, `password`, and `driver_code`. It manually verifies that `driver_code` exists in the `drivers` table and hasn't been claimed (i.e., `auth_id` is null). Then it calls `supabase.auth.signUp()`, followed by an update to `drivers.auth_id`.
- **Login**: Requires `email` and `password`. Calls `supabase.auth.signInWithPassword()`.

## 5. Current RLS / Ownership Model
**RLS Evidence (`src/db/migrations/001_...`, `002_...`)**:
- RLS is explicitly **enabled** on `drivers`, `trips`, and `events`.
- However, **no policies** are defined for the public or authenticated roles.
- **Effect**: All direct client-side database queries are denied. 
- **Ownership**: The app strictly enforces ownership within the server API routes. The route retrieves `user.id` from the secure cookie and queries the DB using `supabaseServer` (which bypasses RLS) to ensure the driver can only interact with their own `trip_id` and `event` records.

## 6. Current Login/Signup/Logout Flow
- **Signup**: Email + Password + Pre-existing Driver Code.
- **Login**: Email + Password.
- **Logout**: Handled via `/api/auth/logout` clearing the session cookie.

## 7. Current Driver ID Handling
`driver_code` acts as the Driver ID. It is currently treated as a predefined constraint. A user cannot sign up unless an administrator has first created a row in `drivers` with a specific `driver_code`.

## 8. Current Email Handling
Currently, email delivery relies entirely on Supabase's default, built-in email service (for confirmation/verification emails if enabled).
- **Gap**: Supabase's default email service has a strict rate limit and does not natively support injecting a dynamically generated `driver_code` from the `drivers` table into the auth welcome email.
- **Future Requirement**: Sending the newly generated Driver ID to the user will require either setting up a custom SMTP provider (like Resend) integrated via edge functions/webhooks, or altering the signup flow so the Driver ID is displayed immediately in the UI post-signup.

## 9. Security Findings
- **Driver ID Login Proxy**: A server-side bridge mapping `driver_code` + `password` to an internal `email` + `password` Supabase login is **highly secure**, provided the email mapping is strictly server-side and never leaks the email address to the client.
- **Driver ID Enumeration**: The bridge route must be careful not to differentiate between "Invalid Driver ID" and "Invalid Password" to prevent attackers from enumerating valid Driver IDs.
- **Data Isolation**: Because RLS denies all client-side access, and server API routes securely map `auth.users.id` to `drivers.auth_id`, it is impossible for a driver to access another driver's trips or events by manipulating URLs or payloads.
- **Secrets**: No service-role credentials (`SUPABASE_SERVICE_ROLE_KEY`) are exposed to the browser. They are safely used only in `src/lib/supabase-server.ts`.

## 10. Target Design vs Current Design
| Feature | Current Design | Target Design |
|---|---|---|
| **Identity** | Email | Driver ID (`driver_code`) |
| **Login Input** | Email + Password | Driver ID + Password |
| **Signup Input** | Email + Password + predefined `driver_code` | Email + Password |
| **ID Generation** | Pre-populated by Admin | Generated by System at Signup |

## 11. Gaps / Risks
1. **Email Customization**: Sending the Driver ID via email is not easily done with out-of-the-box Supabase Auth. It will require a webhook/trigger and a custom mailer (e.g. Resend) or displaying it directly in the UI.
2. **Login Proxy**: Supabase does not natively support logging in with an arbitrary metadata field (`driver_code`). The server route must securely fetch the email associated with the `driver_code` before calling Supabase Auth. Since the `drivers` table doesn't store the email, the server must query `auth.users` via the `drivers.auth_id` link.

## 12. What Can Be Reused
- The `drivers`, `trips`, and `events` database schemas remain perfectly intact.
- The entire `Arrival → Check-in → Departure → Timeline → AI Evidence Summary` flow requires **zero modifications**. The `user.id` to `drivers.auth_id` resolution logic seamlessly handles the authentication redesign.
- The `@supabase/ssr` cookie-based session architecture.

## 13. What Must Change Later
- `/api/auth/signup/route.ts` must be refactored to auto-generate the `driver_code` (e.g., `DRV00X`), create the `drivers` record automatically, and handle notifying the user.
- `/api/auth/login/route.ts` must be refactored to accept `driver_code`, look up the associated `auth_id`, fetch the associated `email` from `auth.users`, and then proxy the login to Supabase.
- Login and Signup UI forms in `src/app/login` and `src/app/signup`.

## 14. Recommended Architecture (Investigation Conclusion)
The architecture requested in the prompt is **correct and safe**:
```text
UI (Driver ID + Password) 
  → Server Route 
  → Lookup driver_code in `drivers` to find `auth_id` 
  → Lookup `auth.users` to find `email`
  → Authenticate with Supabase Auth (Email + Password) 
  → Return Session
```

## 15. Recommended Implementation Sequence
1. Modify `src/app/api/auth/signup/route.ts` and `src/app/signup/page.tsx` to remove `driver_code` input and auto-generate it.
2. Modify `src/app/api/auth/login/route.ts` and `src/app/login/page.tsx` to accept `driver_code` instead of email, performing the server-side lookup bridge.
3. Determine if the Driver ID should be displayed in the UI post-signup (MVP approach) or if a Resend SMTP integration is strictly required for email delivery.

## 16. Files Likely Requiring Changes
- `src/app/login/page.tsx`
- `src/app/signup/page.tsx`
- `src/app/api/auth/login/route.ts`
- `src/app/api/auth/signup/route.ts`

## 17. UNKNOWN / Evidence Still Needed
- **UNKNOWN**: How the project intends to send emails. If Supabase's built-in service is strictly required, we cannot easily inject `driver_code`. An external provider (like Resend) or an MVP fallback (showing the code on-screen) must be decided.
