# Chat12 — Node 2 Investigation: Q5 Authentication Rate Limiting

## 1. Executive Conclusion
The recommended Q5 policy is a **Hybrid of Policy C (Platform-only controls) for the MVP, evolving to Policy B (Layered IP + Account controls) for production.** Supabase GoTrue already provides robust, built-in rate limiting for signups, logins, and email sending. Relying on Supabase's platform controls prevents reinventing the wheel and avoids dangerous account-lockout DoS vectors, while ensuring generic 429 responses. 

## 2. Evidence
- **Verified Supabase/platform evidence:** Supabase Auth natively implements rate limits on `/token` (login), `/signup`, and `/recover` endpoints. It natively limits email sending (e.g., 3 emails per hour per address) to prevent email bombing.
- **Security best-practice evidence:** Hard account lockouts (Policy D) allow malicious actors to trivially DoS legitimate users by spamming incorrect passwords. Rate limiting (429) or progressive delays are the industry standard.
- **Inference:** Because our Next.js API routes (`/api/auth/signup` and `/api/auth/login`) act as proxies to Supabase, they inherit Supabase's backend rate limits, provided the IP addresses are forwarded correctly.

## 3. Threat Model
- **Brute force & Credential stuffing:** Prevented by Supabase's IP and account rate limits on the `/token` endpoint.
- **Email bombing (Signup / Reset):** Prevented by Supabase's native email throttling (typically 3-4 emails per hour).
- **Account lockout DoS:** Mitigated by rejecting hard lockouts. We use HTTP 429 instead, meaning the account remains accessible to the legitimate user if they try from a clean IP.
- **Enumeration:** Supabase returns generic error messages (e.g., "Invalid login credentials") regardless of whether the email exists.

## 4. Candidate Comparison
- **Policy A (IP-only):** Ineffective against distributed botnets (credential stuffing).
- **Policy B (Layered IP + Account):** Ideal enterprise state. Limits per IP to stop botnets, and per Account to stop targeted brute force.
- **Policy C (Platform-only):** **Recommended for MVP.** Supabase already implements Policy B internally.
- **Policy D (Hard Account Lockout):** Rejected due to DoS risk.

## 5. Recommended Architecture
```text
Client Request
 ↓
Next.js API Route (/api/auth/*)
 ↓ (Optionally add Upstash Rate Limit here in future)
Supabase Auth (GoTrue)
 ↓ (GoTrue evaluates IP & Email rate limits)
If Limit Exceeded → GoTrue returns 429 Too Many Requests
If Valid → GoTrue processes authentication
 ↓
Freight Identity/Active Gate (Q2/Q3) evaluated by application
```

## 6. Threshold Policy
Rely on Supabase Default Configuration for MVP:
- **Email OTP/Links:** ~3-4 per hour per email.
- **Signups:** ~30 per hour per IP.
- **Token requests (Login):** ~30 per hour per IP.
*If a custom Upstash limiter is added to Next.js:* limit to 5 login attempts per 15 minutes per IP/Email.

## 7. State / Behavior Matrix

| Endpoint | Scenario | Limiter State | Authentication Result | Response | Account Existence Leaked? |
|---|---|---|---|---|---|
| Login | normal valid attempt | under limit | provider handles | 200 OK | NO |
| Login | repeated failures | limit exceeded | blocked | 429 Too Many Requests | NO |
| Login | many IPs targeting one account | account limit exceeded | blocked | 429 | NO |
| Signup | repeated attempts | limit exceeded | blocked | 429 | NO |
| Password reset | normal request | under limit | provider handles | 200 OK | NO (Generic success) |
| Password reset | repeated request | limit exceeded | blocked | 429 | NO |
| Email resend | repeated request | limit exceeded | blocked | 429 | NO |
| Any | limiter unavailable | fails open (Supabase handles) | provider handles | Varies | NO |

## 8. Security Analysis
- **Fail-open vs Fail-closed:** If a custom application-level limiter (e.g., Redis) goes down, it should **fail-open** and let the request proceed to Supabase, relying on Supabase's built-in limits as a fallback, ensuring the application doesn't go entirely offline.
- **Enumeration:** Generic 429 responses and generic "Password reset email sent (if account exists)" messages prevent enumeration.
- **Distributed Attacks:** Supabase's internal heuristics and WAF handle basic distributed attacks. Advanced botnets require Cloudflare Turnstile or similar.

## 9. Q2 / Q3 Interaction
- **Q2 (Active Status):** Rate limiting happens *before* Q2. A rate-limited user cannot log in, so they cannot even be evaluated for Q2 Active status.
- **Q3 (Session Lifecycle):** Rate limiting applies to *establishing* the session. Once a session is established, token refreshes should have very generous limits to prevent logging out legitimate users.
- **Node 1 Authorization:** Irrelevant to authentication rate limiting. Node 1 only executes if a valid, non-rate-limited session is successfully authenticated.

## 10. Contract Changes Required
In `Chat11_Node2_Authentication_Identity_Contract_DRAFT.md`:
- Document reliance on Supabase's native GoTrue rate limiting for the MVP.
- Explicitly reject Hard Account Lockouts in favor of HTTP 429 responses.
- Mandate generic responses to prevent email enumeration.

## 11. Acceptance Tests
- **IP burst limit:** Fire 50 login requests from one IP -> Receive 429.
- **Password reset abuse:** Request 5 password resets for the same email -> Receive 429.
- **No account enumeration:** Attempt password reset on non-existent email -> Receive generic success message (no leakage).
- **Legitimate login:** Wait out the 429 cooldown, attempt valid login -> Succeeds (Q2/Q3 logic executes normally).

## 12. Remaining Unknowns
- Exactly how the Next.js backend forwards client IPs to Supabase (needs `x-forwarded-for` header config to ensure Supabase doesn't rate-limit the Next.js server itself).

## 13. Implementation Status
`Implementation authorization = NOT GRANTED`
