# Chat8 — Node 3 Report: Rate Limiting for Next.js → Supabase Auth

## 1. Executive Conclusion
The current authentication architecture introduces a significant Denial of Service (DoS) vulnerability. Because Next.js server API routes currently proxy login requests to Supabase Auth without forwarding the original client IP or implementing application-level rate limits, all authentication attempts appear to originate from Vercel's IP addresses. A single attacker attempting to brute-force a login could trigger Supabase's native rate limits, thereby locking out the Vercel server and preventing all legitimate users from logging in. 

To mitigate this, Freight must implement **Application-Level Rate Limiting (Option C)** within Next.js using a dual-key strategy (Per-IP and Per-Driver ID buckets) before the request ever reaches Supabase.

**Decision: APPROVE** layered protection with Next.js application-level rate limiting.

## 2. Current Login Architecture Findings
- **Login API:** `src/app/api/auth/login/route.ts`.
- **Flow:** `Browser -> Vercel (Next.js API route) -> Supabase Auth`.
- **IP Forwarding:** A search of the `api/auth` directory confirms that NO client headers (such as `x-forwarded-for`) are currently being extracted or forwarded to the Supabase client.
- **Rate Limiting:** No custom rate-limiting or bot protection is implemented in the repository.
- **Error Handling:** Currently returns generic `Invalid Driver ID or password` responses, which correctly mitigates account enumeration.

## 3. Current Supabase Rate-Limit Behavior
- Supabase Auth implements native IP-based token-bucket rate limiting to protect endpoints like `signInWithPassword`.
- The default limit is generally 30 requests per minute (token bucket, refilling over time).
- When the limit is exceeded, Supabase returns a `429 Too Many Requests` error.
- Because Vercel does not forward the client IP, Supabase tracks the Vercel server's IP in its rate-limit bucket. If the bucket is exhausted by an attacker, Supabase will block Vercel, causing a site-wide authentication outage.

## 4. Current IP Forwarding Requirements
- Supabase Auth can accept forwarded IPs if configured correctly.
- However, relying solely on Supabase to parse `x-forwarded-for` from an anon-key request can be problematic, as allowing anon clients to define their own IPs introduces spoofing vulnerabilities. While Supabase handles this via trusted proxies, abstracting rate limits to a 3rd-party SaaS behind a Serverless compute layer is fundamentally risky for DoS.

## 5. Next.js/Vercel Trusted-IP Analysis
- In Vercel, the original client IP is reliably injected into the `x-forwarded-for` header.
- Vercel strips user-spoofed `x-forwarded-for` headers at their edge proxy and replaces them with the verified client IP. Therefore, within the Next.js API route (`req.headers.get('x-forwarded-for')`), this header can be trusted.

## 6. Attack-Type Analysis
- **Brute force (Many passwords, 1 Account):** Defeated by Per-Account rate limiting.
- **Password spraying (Few passwords, Many Accounts):** Defeated by Per-IP rate limiting.
- **Credential stuffing:** Defeated by Per-IP rate limiting and random Driver IDs.
- **Distributed attacks (Many IPs):** Supabase/Vercel native protections offer some mitigation, but advanced bot protection/CAPTCHA is eventually required.

## 7. Supabase-Only vs Layered Protection Comparison
- **Option A (Supabase Only):** Fails. Causes site-wide DoS because Vercel IPs get blocked.
- **Option B (Supabase + IP Forwarding):** Better, but brittle. Relies on Supabase trusting the headers and doesn't stop Vercel compute usage (costly).
- **Option C (Next.js App-Level Rate Limiting + Supabase):** **Recommended**. Stops abuse at the Vercel Edge/Serverless layer before it consumes Supabase resources. Highly customizable.
- **Option D (Option C + CAPTCHA):** Overkill for the current 22-day MVP phase.

## 8. Recommended Rate-Limit Key Strategy
**Dual-Bucket Strategy:**
1. **Per-IP Bucket:** Keyed by the extracted `x-forwarded-for` header. Limit: e.g., 20 attempts per minute. (Prevents password spraying and bot abuse).
2. **Per-Account Bucket:** Keyed by the provided `Driver ID`. Limit: e.g., 5 attempts per 15 minutes. (Prevents targeted brute-force).

*Why both?* A single bucket keyed by `IP + Driver ID` allows an attacker to password-spray 10,000 Driver IDs from a single IP, completely bypassing the limit.

## 9. Recommended Account-Lockout/Throttling Strategy
- **Progressive Delays/Temporary Throttling:** The rate limiter should return a `429 Too Many Requests` error when buckets are exhausted, temporarily blocking requests for a window (e.g., 15 minutes).
- **Avoid Hard Lockout:** Do not permanently lock an account or require admin intervention. A malicious user could trivially DoS legitimate drivers by intentionally failing logins against their Driver IDs. Temporary throttling is safe and effective.

## 10. CAPTCHA/Bot-Protection Assessment
- Not recommended for the MVP. Implementing CAPTCHA introduces significant UX friction and implementation complexity.
- A well-configured dual-bucket rate limiter provides sufficient protection for a hackathon/MVP stage.

## 11. Enumeration/Side-Channel Analysis
- The rate limiter must increment the `Driver ID` bucket *regardless* of whether the Driver ID actually exists in the database. 
- If the system only rate-limits *valid* Driver IDs, an attacker can enumerate the database: if they get a `401 Unauthorized`, the ID doesn't exist; if they get a `429 Too Many Requests`, the ID exists. 
- The rate limiting must occur *before* the database lookup.

## 12. Logging/Monitoring Recommendation
- Log all `429 Too Many Requests` events.
- The log should include: Timestamp, IP Address, `Driver ID` attempted, and which bucket triggered the limit (IP or Account).
- **NEVER** log the submitted password.

## 13. Interaction with Random Driver IDs
- Random Driver IDs (from the previous investigation) stop attackers from easily guessing valid usernames.
- However, if an attacker obtains a valid Driver ID (e.g., shoulder surfing), rate limiting remains the only defense against brute-forcing the password. The two controls are complementary.

## 14. Future RBAC Implications
- Rate limits can be adjusted based on roles later (e.g., stricter limits for Admins). However, since rate limiting happens before authentication, we cannot reliably know the user's role during a failed login attempt. The base limits should be conservative enough to protect all roles.

## 15. Domain Dependency Assessment
- Application-level rate limiting (e.g., using `@upstash/ratelimit` or a custom Redis implementation) works seamlessly on the Vercel default domain. No custom domain is required.

## 16. Recommended Implementation Priorities

### Must Implement Now (High Priority)
- Forward the client IP to the Supabase client OR implement a basic Next.js rate limiter to prevent the Vercel IP from being blacklisted by Supabase.

### Should Implement During Current Project (Medium Priority)
- Implement a robust dual-bucket (Per-IP and Per-Driver ID) rate limiter in Next.js using a fast data store (like Redis/Upstash) prior to launching.
- Apply the rate limiter to both `/api/auth/login` and `/api/auth/signup`.

### Future Production Hardening
- Implement a CAPTCHA or invisible bot protection challenge (e.g., Turnstile) when the rate limiter detects suspicious activity.

## 17. Exact Source Files/Configuration Areas to Modify
- `src/app/api/auth/login/route.ts`: Insert rate-limiting middleware/logic before the database lookup.
- `src/app/api/auth/signup/route.ts`: Insert rate-limiting logic.
- `src/lib/rate-limit.ts` (NEW): Create a utility for the rate-limiting logic (e.g., wrapping Upstash Redis).

## 18. Verification/Test Plan
- Attempt to log in >20 times from the same IP (using a script) and verify the server returns a `429` status code and does not hit the Supabase backend.
- Attempt to log in to the *same* Driver ID >5 times from different IPs (simulate via headers if possible) and verify the account bucket triggers a `429`.
- Verify that Supabase dashboard logs do not show the Vercel server IP being rate-limited.

## 19. Unresolved Questions/Blockers
- Does the project owner want to provision a Redis instance (e.g., Vercel KV / Upstash) for rate limiting, or use a simpler in-memory store (which is less effective in serverless environments but requires no setup)? 

## 20. Final Recommendation
**APPROVE**. Implement Next.js application-level rate limiting using a dual-bucket strategy (IP and Driver ID) to protect both the user accounts and the Vercel infrastructure.
