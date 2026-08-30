# Chat17 — Day 10 — Password Recovery Fresh-Link Failure Investigation Report

## 1. Investigation Status
COMPLETED

## 2. Observed Failure
- Ayush generated a fresh password recovery email and clicked the newest link once.
- The browser was redirected to `/login?error=Invalid+or+expired+recovery+link`.
- The URL briefly contained: `error=access_denied&error_code=otp_expired&error_description=Email+link+is+invalid+or+has+expired`.

## 3. Evidence Inspected
- **URL Error Parameters**: The presence of `error_code=otp_expired` in the query string indicates that Supabase Auth rejected the token *before* our application logic could establish a session.
- **Route Implementation**: `src/app/api/auth/confirm/route.ts` is implemented using the `token_hash` and `type` Email OTP flow (`supabase.auth.verifyOtp`), not the PKCE `code` flow (`exchangeCodeForSession`).
- **Fallback Logic**: The `confirm` route falls back to redirecting to `/login?error=...` if `token_hash` and `type` are not present, completely ignoring any error query parameters sent by Supabase.

## 4. Exact Root Cause
There are two intersecting root causes for this failure:

1. **Email Scanner Pre-fetching (The `otp_expired` source)**:
   - The Supabase project is currently using the default email template variable `{{ .ConfirmationURL }}`. This link points directly to the Supabase Auth server's `/verify` GET endpoint.
   - Enterprise email security scanners (or standard client pre-fetching) crawl this link as soon as the email is received.
   - The HTTP GET request by the scanner consumes the single-use OTP.
   - When the actual user clicks the link seconds later, Supabase sees the OTP is already consumed, rejects it with `otp_expired`, and redirects to the configured app URL with the error parameters.
2. **Configuration/Implementation Mismatch**:
   - Even if a scanner did not consume the link, the flow would still fail.
   - The default `{{ .ConfirmationURL }}` template uses the PKCE flow and redirects the user to the application with a `?code=xxxx` query parameter.
   - However, our `src/app/api/auth/confirm/route.ts` is explicitly checking for `token_hash` and `type`. Because `token_hash` is undefined in the PKCE redirect, the route bypasses verification and immediately drops the user at the `/login` fallback.

## 5. Minimal Corrective Decision
To resolve both the scanner consumption issue and the parameter mismatch, we must align the configuration and source code to use a secure, scanner-resistant Email OTP flow.

**1. Configuration Change (Supabase Dashboard)**:
Change the Password Recovery email template to point directly to the application's client-side update-password page, passing the `token_hash` in the URL hash fragment or query parameters.
- **Action**: Update the Supabase Password Recovery template to:
  `Click here to reset your password: <a href="{{ .SiteURL }}/update-password?token_hash={{ .TokenHash }}&type=recovery">Reset Password</a>`

**2. Source Code Change (Application)**:
Bypass the server-side GET `/api/auth/confirm` route entirely for password recovery, as any GET API route remains susceptible to scanner pre-fetching.
- **Action**: Update `src/app/update-password/page.tsx` (a client component) to read `token_hash` and `type` from the URL. On mount (or upon clicking "Update Password"), the client uses `@supabase/ssr` or `@supabase/supabase-js` to call `supabase.auth.verifyOtp()`. Since email scanners do not execute complex Single Page Application JavaScript, the token will survive until the real user opens the page.
- **Action**: Remove or deprecate `/api/auth/confirm` for password recovery.

## 6. Security Implications
- Moving the OTP verification to a client-side component (or requiring a POST interaction) is the industry standard for mitigating email scanner link consumption.
- This change maintains the strict separation of authentication and authorization, and fully preserves the Node 1/Node 2 identity rules.
