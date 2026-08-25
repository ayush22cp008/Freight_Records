# Chat12 — Node 2 Investigation: Q3 Session Lifecycle & Refresh

## 1. Executive Conclusion
The recommended Q3 policy is **Policy B (Middleware-centered session refresh)**. Given the Next.js App Router architecture and `@supabase/ssr` package, the most secure and robust pattern is to use Next.js Middleware to read the session cookies, call `supabase.auth.getUser()`, handle automatic refresh token rotation, and append the updated cookies to the response. Protected API routes and Server Components should subsequently rely on the refreshed session and additionally verify the user's `Active` status (email confirmed + verified).

## 2. Evidence
- **Verified Supabase/platform evidence:** Supabase uses short-lived JWT access tokens and long-lived refresh tokens. Refresh token rotation is enabled by default to prevent replay attacks.
- **Verified repository evidence:** The `@supabase/ssr` library provides `createServerClient` which must be used in Next.js middleware to securely write updated cookies back to the client during a refresh.
- **Inference:** Server components in Next.js cannot modify cookies on the response. Therefore, session refresh MUST happen in Middleware or API routes to keep the browser cookies synchronized.

## 3. Session Lifecycle Diagram
```text
Login → Set HTTP-Only Cookies (Access + Refresh Token)
 ↓
Protected Route Requested
 ↓
Middleware intercepts → calls supabase.auth.getUser()
 ↓
If Access Token expired → Supabase automatically uses Refresh Token
 ↓
Middleware writes new Access/Refresh tokens to Response Cookies
 ↓
Server Component/API receives valid session → checks Q2 Active Gate
 ↓
Node 1 Authorization → Business Operation
```

## 4. State Matrix

| Session | Email | Verification | Trusted Role | Active | Protected Access | Expected Behavior |
|---|---|---|---|---|---|---|
| None | Confirmed | VERIFIED | Driver/Company | YES | NO | Redirect to login / 401 |
| Valid | Confirmed | PENDING | NULL | NO | NO | Session valid, but 403 on business operations |
| Valid | Confirmed | VERIFIED | Driver/Company | YES | YES | Fully active, 200 OK |
| Valid | Unconfirmed | VERIFIED | Driver/Company | NO | NO | Session valid, but 403 (unconfirmed email) |
| Expired | Confirmed | VERIFIED | Driver/Company | YES | YES* | *Refreshed seamlessly by middleware, then 200 |
| Invalid | Confirmed | VERIFIED | Driver/Company | YES | NO | Session rejected, clear cookies, redirect/401 |
| Valid | Confirmed | REJECTED | NULL | NO | NO | Session valid, but 403 permanently |

## 5. Candidate Comparison
- **Policy A (Server-verified only):** Flawed in Next.js App Router because Server Components cannot set cookies. Expired tokens cannot be refreshed cleanly.
- **Policy B (Middleware-centered refresh):** **Recommended.** Middleware can refresh tokens and write cookies to the `NextResponse` before the Server Component runs.
- **Policy C (Client-only):** Rejected. Insecure for SSR applications. Susceptible to token exposure and lacks strict server-side enforcement.

## 6. Security Analysis
- **Stolen access tokens:** Mitigated by short JWT expiry (e.g., 1 hour).
- **Refresh token replay:** Mitigated by Supabase's automatic refresh token rotation. Reused tokens revoke the entire token family.
- **Stale JWT claims:** Supabase JWTs contain the `email_verified` claim. If a user changes their email, `email_verified` becomes false. Protected routes MUST verify this claim directly rather than caching it, preventing stale access.
- **Browser token exposure:** Tokens are stored in HttpOnly, Secure, SameSite=Lax cookies, preventing XSS extraction.
- **Logout:** Explicit logout (`supabase.auth.signOut()`) revokes the session on the GoTrue server and clears local cookies.

## 7. Q2 Interaction
Session validity is completely decoupled from the Active status:
- A user can hold a perfectly **Valid Session** while being `PENDING`, `REJECTED`, or `Unconfirmed`.
- The session proves *who* the user is, but the **Q2 Active Gate** proves *what they are allowed to do*.
- Every protected server operation must check both the session's validity AND the `verification_status`/`email_confirmed` fields before granting access.

## 8. Recommended Q3 Decision
`Q3 = Policy B (Middleware-centered session refresh)`
- **Establishment:** `signIn` sets HttpOnly cookies.
- **Validation:** Middleware validates via `getUser()`.
- **Refresh:** Handled automatically by `getUser()` in Middleware, updating cookies.
- **Refresh Failure / Invalid:** Cookies cleared, user redirected to `/login` (or 401 for APIs).
- **Logout:** `signOut()` revokes tokens server-side and clears cookies.
- **Interaction with Q2:** The Active gate must be evaluated independently on every protected request.

## 9. Contract Changes Required
In `Chat11_Node2_Authentication_Identity_Contract_DRAFT.md`:
- Mandate the use of Next.js Middleware for centralized session refresh.
- Explicitly decouple Session Validity from Active Account Status.
- Define HTTP-Only cookies as the strict storage boundary.

## 10. Acceptance Tests
- **Valid protected request:** User with active status accesses route -> Succeeds.
- **Expired access token:** Wait 1 hour, access route -> Token is refreshed seamlessly by middleware, access succeeds.
- **Failed refresh:** Corrupt the refresh token cookie, access route -> Redirected to login.
- **Logout:** Call logout, access route -> Redirected to login.
- **Email change:** User verified and active, changes email (unconfirmed). JWT claims `email_verified=false`. Attempt to access protected route -> Fails (403).

## 11. Remaining Unknowns
- None.

## 12. Implementation Status
`Implementation authorization = NOT GRANTED`
