# Chat11 — Node 2 Authentication + Identity Investigation — Round 2

## 1. Protected route and API authentication
- **UI Protection:** Authenticated UI routes are protected by a layout guard in `src/app/(authenticated)/layout.tsx`. It calls `await createClient().auth.getUser()` and redirects to `/login` if `data?.user` is not found.
- **API Protection:** API routes (e.g., `src/app/api/events/arrival/route.ts`) check authentication server-side via `await createClient().auth.getUser()`. If no user is found, they return a 401 error. 
- **Middleware:** No root `middleware.ts` or similar catch-all route guard was found. The protection relies entirely on the Next.js `layout.tsx` for UI and individual manual checks in each API route.
- **Client-Side vs Server-Side:** All protections found are executed server-side (in Server Components and API Routes).

## 2. Authenticated request context
- **Available Context:** After authentication, the server only obtains the raw `auth.users` object via `getUser()`, which provides `user.id` and `user.email`.
- **Identity Retrieval:** API routes (like `arrival/route.ts`) manually fetch the Driver identity by querying `supabaseServer.from('drivers').select('id').eq('auth_id', user.id).single()`.
- **Missing Context:** Company identity and Role are not available anywhere in the context or the database. There is no unified application identity identifier passed in the session.

## 3. Session and cookie behavior
- **Cookie Creation/Handling:** Sessions are managed using standard Supabase defaults via `@supabase/ssr`. `src/lib/supabase/server.ts` defines `createClient` which reads/writes cookies using `next/headers`.
- **Session Refresh Issue:** The `setAll` method in `src/lib/supabase/server.ts` notes that it cannot set cookies when called from a Server Component. Without a `middleware.ts` to refresh user sessions, the access token will likely fail to refresh automatically during Server Component rendering if it expires.
- **Tokens/Storage:** Browser storage relies on standard Supabase cookies (typically `sb-[ref]-auth-token`). Access/refresh tokens are handled by the Supabase SDK automatically.

## 4. Signup and email verification
- **Email Confirmation:** The code in `src/app/api/auth/signup/route.ts` creates the user via `signUp` and immediately proceeds to create the Driver profile. It does not check if email confirmation is required or block the driver creation if unconfirmed.
- **Atomicity:** Account creation is **not** atomically coupled. The `signUp` call happens first. If successful, it performs a separate `insert` into the `drivers` table. If the `drivers` insert fails, the `auth.users` record remains orphaned, leading to inconsistent state.
- **Login for Unconfirmed Users:** The login route uses `supabase.auth.signInWithPassword`. Whether it allows unconfirmed users depends on the external Supabase project settings, but the application code does not contain explicit logic to handle unconfirmed states gracefully.
