# Chat12 — Node 2 Investigation: Q5 Authentication Rate Limiting

**Status:** 🟡 INVESTIGATED / NOT LOCKED  
**Implementation authorization:** ❌ NOT GRANTED

## 1. Executive Conclusion

### Recommended MVP policy

**Use Supabase Auth's native authentication rate limiting as the primary abuse-control mechanism for the MVP. Do not add a custom Redis/Upstash rate limiter unless new evidence shows that the application's proxy architecture creates a material protection gap.**

Do **not** describe Supabase's built-in controls as an exact implementation of Freight's own production Policy B. Supabase provides endpoint-specific rate limits using different dimensions and mechanisms. A future production hardening phase may add a layered application-level limiter if the threat model and deployment architecture require it.

Final candidate policy:

```text
Q5 MVP
= Supabase-native Auth rate limiting
+ no hard account lockout
+ generic authentication responses
+ correct client-IP forwarding where required
+ 429 handling
+ no custom distributed limiter unless evidence requires it
```

## 2. Evidence

### Verified Supabase evidence

Current official Supabase documentation confirms:

- Auth enforces rate limits on authentication endpoints.
- IP-limited operations use a token-bucket algorithm; a bucket can permit brief bursts before sustained traffic is throttled.
- Exceeding a rate limit produces HTTP `429 Too Many Requests`.
- Signup/sign-in requests have a documented IP-based quota.
- Token refresh, verification, MFA challenge/verify, and other Auth operations have their own endpoint-specific limits.
- Email sending has separate project-wide and per-user cooldown controls depending on the endpoint and configuration.
- Supabase supports client-IP forwarding for server-side frameworks/proxies using the `Sb-Forwarded-For` header, when IP forwarding is explicitly enabled and the request uses a supported secret API key.

Source used for current platform behavior:

`https://supabase.com/docs/guides/auth/rate-limits`

The exact values are platform configuration details and may change. The project should not hard-code current Supabase defaults into the Node 2 architecture contract unless the application intentionally configures and owns those values.

### Verified repository evidence

The current application architecture uses Next.js authentication routes that proxy authentication operations to Supabase. Therefore, the investigation must verify the actual deployed request path and IP-forwarding behavior before claiming that Supabase sees the original client IP.

### Security best-practice evidence

Hard account lockout is rejected as the primary brute-force control because an attacker can intentionally cause denial of service against a known victim account. Rate limiting and generic authentication responses provide safer abuse control.

### Inference

For the hackathon MVP, relying on Supabase's existing Auth controls is lower-risk and simpler than introducing a new distributed rate-limiting dependency.

## 3. Threat Model

Relevant threats:

- password brute force;
- credential stuffing;
- repeated login attempts against one account;
- many accounts targeted from one IP;
- distributed login attempts across many IPs;
- signup abuse;
- password-reset email abuse;
- confirmation/OTP resend abuse;
- authentication resource exhaustion;
- account enumeration;
- account-lockout denial of service.

Rate limiting materially helps with burst/sustained request abuse, but it does not by itself solve every distributed-botnet or credential-stuffing scenario. Additional bot protection such as CAPTCHA/Turnstile may be considered later if the threat model requires it.

## 4. Candidate Policy Comparison

| Policy | Assessment | Decision |
|---|---|---|
| A — IP-only custom limiter | Useful but incomplete against distributed attacks and creates new infrastructure | Not selected |
| B — Layered IP + account/identifier + endpoint custom limiter | Stronger production architecture but adds shared-state infrastructure and complexity | Future option |
| C — Supabase-native Auth controls | Already present, endpoint-specific, low implementation complexity | **SELECTED for MVP** |
| D — Hard account lockout | Creates account-lockout DoS risk | **Rejected** |

Important correction:

```text
Supabase native controls
≠
Freight's complete custom Policy B
```

Supabase provides substantial built-in protection, but the exact limiting dimension varies by Auth operation. Freight should not claim that every operation is simultaneously limited by both IP and account.

## 5. Recommended Architecture

```text
Client
   ↓
Next.js authentication route / Supabase Auth request
   ↓
Supabase Auth native rate limiting
   ↓
If quota exceeded → 429
   ↓
If allowed → authentication operation
   ↓
Session established/updated
   ↓
Q3 session lifecycle
   ↓
Q2 Active gate for protected business access
   ↓
Node 1 authorization
```

### Important proxy/IP requirement

Because server-side frameworks can hide the original client IP behind the server/proxy, the actual deployment path must be verified.

If Freight needs Supabase Auth to rate-limit using the end-user IP, the current Supabase mechanism is:

```text
Client IP
   ↓
Next.js / trusted proxy
   ↓
Sb-Forwarded-For
   ↓
Supabase Auth
```

Supabase currently requires IP forwarding to be explicitly enabled and the forwarded request to use a supported secret API key. The exact implementation must be verified against the actual Freight deployment before any change is authorized.

Do not blindly copy an `x-forwarded-for` value from an untrusted client into `Sb-Forwarded-For`.

## 6. Threshold Policy

### MVP

Freight should **rely on Supabase's configured Auth rate limits rather than hard-coding guessed values in application code**.

Current official documentation gives examples such as:

- sign-in/sign-up: IP-based quota of 30 requests per 5-minute interval by default;
- token refresh: IP-based 1800 requests/hour;
- verification requests: IP-based 360 requests/hour;
- built-in email provider: 2 emails/hour project-wide;
- signup confirmation and password-reset requests: 60-second per-user cooldown by default;
- OTP: endpoint-specific configurable limits.

These values are platform defaults/configuration details and must be re-verified at implementation time.

### Future custom limiter

If evidence later justifies an application-level limiter, thresholds must be selected from the actual threat model and deployment pattern. Do not lock an arbitrary value such as `5 attempts / 15 minutes` into the architecture without evidence.

## 7. State / Behavior Matrix

| Endpoint | Scenario | Limiter State | Authentication Result | Expected Response | Account Existence Leaked? |
|---|---|---|---|---|---|
| Login | normal valid attempt | under limit | provider processes | normal success | No |
| Login | sustained excessive traffic | limit exceeded | blocked by Auth | 429 | No |
| Login | invalid credentials under limit | under limit | authentication fails normally | generic auth failure | No |
| Login | many IPs targeting one account | platform controls apply according to endpoint behavior | may be allowed or throttled | generic auth result / 429 when quota exceeded | No |
| Signup | repeated attempts | endpoint quota/cooldown | blocked when limit exceeded | 429 or provider-specific response | No |
| Password reset | normal request | under limit | provider processes | generic success behavior | No |
| Password reset | repeated request | cooldown/quota exceeded | blocked/throttled | 429 or provider-specific response | No |
| Email/OTP resend | repeated request | cooldown/quota exceeded | blocked/throttled | 429 or provider-specific response | No |
| Token refresh | normal refresh | under limit | refresh proceeds | normal session response | No |
| Token refresh | excessive traffic | limit exceeded | refresh blocked | 429 | No |
| Custom limiter not present | MVP | Supabase remains primary limiter | provider handles | provider response | No |
| Future custom limiter unavailable | future architecture | depends on explicitly approved policy | must follow separately approved fail behavior | policy-defined | No |

## 8. Fail-Open / Fail-Closed Policy

For the MVP there is no separate custom distributed limiter whose outage must be handled.

Therefore:

```text
Custom limiter unavailable
        ↓
Not applicable to MVP
        ↓
Supabase Auth remains the primary rate-limit boundary
```

If a custom application-level limiter is introduced later, fail behavior must be decided as part of that separate architecture change. Do not automatically declare universal fail-open behavior.

A future fail-open design would preserve availability but reduce the additional application-level abuse control during an outage. A future fail-closed design would strengthen abuse resistance but risks making authentication unavailable when the limiter infrastructure fails.

## 9. Request Ordering

The safe MVP boundary is:

```text
Authentication request
        ↓
Supabase Auth rate limiting
        ↓
Supabase authentication operation
        ↓
Session establishment / refresh
        ↓
Freight identity / Q2 Active gate
        ↓
Node 1 authorization
```

The application must not perform unnecessary account-specific lookups before the abuse-control boundary if doing so can create a resource-exhaustion or enumeration side channel.

## 10. Failure Responses and Enumeration

Rate-limit responses should use `429 Too Many Requests` where the underlying Auth operation returns a rate-limit error.

Authentication failures should remain generic and must not reveal whether an account exists.

Password-reset and similar recovery flows should preserve the existing non-enumerating user experience rather than changing responses based on account existence.

`Retry-After` should be surfaced/handled only where the actual Auth response provides an appropriate retry interval; do not invent a fixed value.

## 11. Account Lockout Policy

### Decision

**Do not implement hard account lockout as the normal brute-force defense.**

Reason:

```text
Attacker knows victim email
        ↓
Repeated bad passwords
        ↓
Hard account lockout
        ↓
Victim denied legitimate access
```

Supabase rate limiting should remain the first-line MVP control.

## 12. Distributed Attack Analysis

The MVP policy materially addresses common bursts and sustained traffic covered by Supabase's endpoint-specific limits.

It does not guarantee protection against every distributed credential-stuffing attack because no single IP limiter can reliably identify a distributed botnet.

If later evidence shows the threat level requires stronger controls, consider:

```text
Application-level layered limiter
+
CAPTCHA / Turnstile
+
WAF / edge controls
+
security monitoring
```

Such additions require a new architecture decision; they are not part of the current MVP implementation authorization.

## 13. Q2 / Q3 Interaction

### Q2

Rate limiting occurs at authentication/abuse-control boundaries. Passing the limiter does **not** grant Active access.

The locked Q2 invariant remains:

```text
ACTIVE
= email_confirmed
AND verification_status = VERIFIED
AND trusted_role IS NOT NULL
```

### Q3

Rate limiting protects authentication and Auth endpoint operations. It does not replace Q3 session validation/refresh.

The locked Q3 separation remains:

```text
Valid Session
    ≠ Active Freight Account
    ≠ Authorized for every operation
```

Token refresh has its own Supabase Auth rate-limit behavior and must not be subjected to an unnecessarily aggressive custom login limit.

### Node 1

Rate limiting does not replace Node 1 authorization. A request that passes rate limiting still requires the normal authentication, Active, and operation-level authorization checks.

## 14. Supabase vs Freight Responsibility

### Supabase owns for MVP

```text
Auth endpoint abuse controls
Auth rate-limit counters
429 rate-limit enforcement
Auth email/OTP cooldowns and quotas
```

### Freight owns

```text
Q2 Active policy
Q3 session lifecycle policy
Node 1 authorization
Generic application responses where required
Correct proxy/IP handling
Application-level security logging/observability
```

### Future

Freight may add an application-level layered limiter if new evidence demonstrates that Supabase's controls are insufficient for the deployed architecture.

## 15. Acceptance Tests

Do not use arbitrary burst counts as deterministic tests because Supabase's IP-based controls use token-bucket behavior and may allow short bursts before sustained traffic is throttled.

### Required tests

1. Verify sustained login traffic beyond the documented/configured quota eventually produces `429`.
2. Verify normal legitimate login succeeds while under the configured quota.
3. Verify invalid credentials return generic authentication errors and do not reveal account existence.
4. Verify repeated password-reset requests respect the documented/configured cooldown/quota.
5. Verify repeated signup requests respect the documented/configured quota/cooldown.
6. Verify repeated confirmation/OTP requests respect the documented/configured quota/cooldown.
7. Verify token refresh is not accidentally subjected to an aggressive custom login limiter.
8. Verify a rate-limited request cannot create a protected session/business operation.
9. Verify a request that passes rate limiting still reaches Q2 Active enforcement where protected access is attempted.
10. Verify Node 1 authorization still runs after authentication/Active checks.
11. Verify password-reset/recovery responses do not disclose whether an account exists.
12. Verify the deployed proxy path preserves the intended client-IP identity for Supabase rate limiting, using the supported `Sb-Forwarded-For` mechanism if required.
13. Verify an untrusted client cannot directly choose the forwarded-IP value used for server-side rate limiting.
14. Verify `429` responses are handled correctly by the application without converting them into misleading authentication or authorization errors.
15. If a future custom limiter is introduced, separately test its concurrency, shared-state, outage, fail-open/fail-closed, and multi-instance behavior.

## 16. Remaining Unknowns / Required Verification

The following are genuine implementation-time verification items, not blockers to the MVP policy:

1. Confirm the exact Next.js → Supabase Auth request path used by the current application.
2. Confirm whether Supabase currently sees the actual end-user IP or the Next.js/proxy IP.
3. If client-IP forwarding is required, verify the secure server-side mechanism using `Sb-Forwarded-For` and a supported secret API key.
4. Confirm the project's currently configured Supabase Auth rate-limit values before implementation/testing.
5. Confirm which authentication endpoints are actually exposed by Freight beyond the currently identified login/signup/recovery flows.

These must be verified before implementation acceptance testing, but they do not justify adding a custom rate-limiter dependency now.

## 17. Contract Changes Required

If Q5 is locked, the Node 2 contract should state:

1. The MVP relies on Supabase Auth's native endpoint-specific rate limiting.
2. Freight does not implement a custom distributed rate limiter unless a new evidence-based architecture decision authorizes it.
3. Hard account lockout is not the normal brute-force control.
4. Authentication/recovery responses must not reveal account existence.
5. `429 Too Many Requests` is the expected rate-limit response where Supabase returns it.
6. The application must correctly handle the actual Next.js/proxy → Supabase IP path.
7. Client-IP forwarding must use the supported secure server-side mechanism if required.
8. Q5 rate limiting is an abuse-control layer and does not replace Q2 Active or Node 1 authorization.
9. Exact platform rate-limit defaults are configuration evidence, not immutable application constants.

Do not modify the Node 2 contract until Q5 is formally locked.

## 18. Final Recommendation

```text
Q5 = SUPABASE-NATIVE AUTH RATE LIMITING FOR MVP

No custom Redis/Upstash limiter initially.
No hard account lockout.
Use generic authentication/recovery responses.
Handle 429 correctly.
Verify secure client-IP forwarding through the actual proxy architecture.
Reassess layered application-level controls only if new evidence requires them.
```

## 19. Final Status

```text
Q1 = 🔒 LOCKED
Q2 = 🔒 LOCKED
Q3 = 🔒 LOCKED
Q4 = 🔒 LOCKED
Q5 = 🟡 INVESTIGATED — READY FOR INDEPENDENT REVIEW
Implementation = ⏸️ PAUSED
```

Next workflow:

```text
Corrected Q5 report
        ↓
Independent Claude review
        ↓
Ayush approval
        ↓
Q5 LOCK
        ↓
Move to Q6
```

**Do not implement Q5 yet.**