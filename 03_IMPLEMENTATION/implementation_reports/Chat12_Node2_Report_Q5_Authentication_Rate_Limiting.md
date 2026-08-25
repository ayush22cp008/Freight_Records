# Chat12 — Node 2 Decision Record: Q5 Authentication Rate Limiting

**Status:** 🔒 LOCKED  
**Implementation authorization:** ❌ NOT GRANTED

## Final Decision

**Q5 = Supabase-native Auth rate limiting for the MVP.**

Freight will use Supabase Auth's built-in, endpoint-specific rate limiting as the primary authentication abuse-control mechanism for the MVP.

```text
Q5 MVP
= Supabase-native Auth rate limiting
+ no custom Redis/Upstash limiter initially
+ no hard account lockout
+ generic authentication/recovery responses
+ correct secure client-IP forwarding where required
+ correct 429 handling
```

A custom layered application-level limiter may be considered later only if new evidence demonstrates a material protection gap in the deployed architecture.

## Claude Independent Review

Claude independently reviewed the Q5 investigation and approved the underlying policy.

Claude verdict:

```text
LOCK Q5 after two factual corrections
```

The required corrections are incorporated in this locked record:

1. The previous `sign-in/sign-up: 30 requests per 5-minute interval` statement is removed. Current Supabase behavior is endpoint-specific; password sign-in flows through `/auth/v1/token`, whose documented IP rate-limit behavior is separate from the earlier incorrect claim. Exact platform values must be re-verified at implementation time.
2. MFA challenge/verify rate limits are explicitly acknowledged as separate Supabase Auth controls. MFA is outside the current MVP scope and must be re-verified if MFA is introduced later.

## Locked Constraints

Do not reopen:

```text
Q1 = LOCKED — atomic Auth User → Freight Identity creation
Q2 = LOCKED — email confirmation before normal authenticated onboarding and Active access
Q3 = LOCKED — Middleware-centered session refresh + live Freight DB Active gate
Q4 = LOCKED — exactly one Freight Identity per Auth User + DB UNIQUE(auth_user_id)
```

## 1. What Q5 Controls

Q5 controls authentication abuse traffic such as:

- password brute force;
- credential stuffing;
- repeated login attempts;
- signup abuse;
- password-reset abuse;
- confirmation/OTP resend abuse;
- authentication resource exhaustion.

Rate limiting is an **abuse-control layer**. It is not an authentication substitute, Active-state mechanism, or authorization mechanism.

## 2. Supabase vs Freight Responsibility

### Supabase owns for MVP

```text
Auth endpoint rate-limit enforcement
Auth rate-limit counters
429 rate-limit responses
Auth email/OTP cooldowns and quotas
Endpoint-specific Auth abuse controls
```

### Freight owns

```text
Q2 Active policy
Q3 session lifecycle policy
Node 1 authorization
Generic application responses where required
Secure proxy/IP handling
Application security logging/observability
```

Freight must not claim that Supabase implements Freight's complete custom production Policy B. Supabase's controls are endpoint-specific and use different limiting dimensions/mechanisms.

## 3. No Custom Distributed Limiter for MVP

Do not add Redis, Upstash, or another custom distributed rate-limit dependency for the MVP unless new evidence demonstrates a material protection gap.

Reason:

```text
Supabase already provides Auth abuse controls
        ↓
Custom limiter adds infrastructure + complexity
        ↓
No demonstrated MVP requirement
        ↓
Do not add it yet
```

If production evidence later requires layered controls, create a new architecture decision rather than silently expanding Q5.

## 4. No Hard Account Lockout

Hard account lockout is explicitly rejected as the normal brute-force defense.

```text
Attacker knows victim email
        ↓
Repeated bad passwords
        ↓
Hard lockout
        ↓
Victim denied legitimate access
```

Rate limiting/429 behavior is preferred because it limits abuse without giving an attacker a simple account-lockout DoS mechanism.

## 5. Threshold Policy

Freight will **not hard-code guessed Supabase rate-limit values into application logic**.

Supabase's current Auth limits are platform/configuration details and can change. Implementation and acceptance testing must verify the current project configuration and current official Supabase behavior.

Important verified behavior:

- IP-limited Auth operations use a token-bucket model, so short bursts can occur before sustained traffic is throttled.
- Exceeding a configured rate limit results in `429 Too Many Requests` where the underlying Auth operation returns the rate-limit response.
- Auth endpoints have separate quotas/cooldowns depending on the operation.
- Email sending and confirmation/recovery flows have their own project/user cooldown behavior.
- MFA challenge/verify has separate Auth rate limiting, but MFA is outside the current MVP scope.

Do not treat any example/default number as a permanent Freight architecture constant.

## 6. Secure Client-IP Handling

The actual Next.js → Supabase Auth request path must be verified before implementation acceptance testing.

If Supabase must rate-limit using the actual end-user IP through a server-side framework/proxy, use the supported secure server-side forwarding mechanism:

```text
Client IP
   ↓
Trusted Next.js / proxy
   ↓
Sb-Forwarded-For
   ↓
Supabase Auth
```

Do **not** blindly copy a client-controlled `x-forwarded-for` value into `Sb-Forwarded-For`.

The actual deployment/proxy architecture must establish the trusted client-IP source before forwarding.

## 7. Authentication Request Flow

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
Session established / updated
   ↓
Q3 session lifecycle
   ↓
Q2 Active gate for protected business access
   ↓
Node 1 authorization
```

Rate-limit approval does not grant business access.

## 8. Failure Responses / Enumeration

When the underlying Auth operation returns a rate-limit response:

```text
429 Too Many Requests
```

Authentication and recovery flows must remain non-enumerating.

Do not reveal whether an email/account exists through different authentication or recovery responses.

`Retry-After` should only be surfaced/handled when the actual underlying response provides an appropriate retry interval; Freight must not invent a fixed retry value.

## 9. Fail-Open / Fail-Closed

For the MVP, there is no separate custom distributed limiter whose outage requires a Freight-specific fail-open/fail-closed policy.

```text
Custom limiter unavailable
        ↓
Not applicable to MVP
        ↓
Supabase Auth remains primary rate-limit boundary
```

If a custom application-level limiter is introduced later, fail behavior must be decided as part of that new architecture decision.

## 10. Q2 / Q3 / Node 1 Interaction

### Q2

Passing the rate limiter does **not** make the user Active.

The locked Q2 invariant remains:

```text
ACTIVE
= email_confirmed
AND verification_status = VERIFIED
AND trusted_role IS NOT NULL
```

### Q3

Rate limiting protects Auth operations; it does not replace session validation or refresh.

The locked Q3 separation remains:

```text
Valid Session
    ≠ Active Freight Account
    ≠ Authorized for every operation
```

Token refresh has its own Auth rate-limit behavior and must not accidentally be subjected to an aggressive custom login limiter.

### Node 1

A request that passes rate limiting must still pass normal authentication, Active, and operation-level authorization checks.

## 11. Security Boundary

```text
Rate limiting
    ↓
Authentication
    ↓
Q3 session lifecycle
    ↓
Q2 Active gate
    ↓
Node 1 authorization
```

No layer may be skipped because another layer succeeded.

## 12. Acceptance Tests

The following are the required Q5 acceptance tests. Tests must use the project's actual Supabase configuration at implementation time rather than relying on guessed default numbers.

1. Sustained authentication traffic beyond the documented/configured quota eventually produces `429`.
2. Normal legitimate login succeeds while under the configured quota.
3. Invalid credentials return generic authentication errors and do not reveal account existence.
4. Repeated password-reset requests respect the configured/documented cooldown/quota.
5. Repeated signup requests respect the configured/documented quota/cooldown.
6. Repeated confirmation/OTP requests respect the configured/documented quota/cooldown.
7. Token refresh is not accidentally subjected to an aggressive custom login limiter.
8. A rate-limited request cannot create a protected session/business operation.
9. A request that passes rate limiting still reaches Q2 Active enforcement when protected access is attempted.
10. Node 1 authorization still runs after authentication/Active checks.
11. Password-reset/recovery responses do not disclose whether an account exists.
12. The deployed proxy path preserves the intended client-IP identity for Supabase rate limiting, using the supported `Sb-Forwarded-For` mechanism if required.
13. An untrusted client cannot directly choose the forwarded-IP value used for server-side rate limiting.
14. `429` responses are handled correctly without converting them into misleading authentication or authorization errors.
15. If a future custom limiter is introduced, separately test its concurrency, shared-state, outage, fail-open/fail-closed, and multi-instance behavior.
16. If MFA is introduced later, verify the applicable Supabase MFA rate limits before enabling/accepting the feature.

## 13. Remaining Implementation Verification

These are implementation-time verification items, not unresolved policy decisions:

1. Confirm the exact Next.js → Supabase Auth request path.
2. Confirm whether Supabase sees the actual end-user IP or the Next.js/proxy IP.
3. If client-IP forwarding is required, verify the secure `Sb-Forwarded-For` mechanism.
4. Confirm the project's configured Supabase Auth rate limits before acceptance testing.
5. Confirm which Auth endpoints are actually exposed by Freight.

These checks do not authorize adding a custom rate limiter.

## 14. Contract Requirements

The Node 2 contract must eventually state:

1. MVP uses Supabase Auth's native endpoint-specific rate limiting.
2. No custom distributed limiter unless a new evidence-based decision authorizes it.
3. Hard account lockout is not the normal brute-force control.
4. Authentication/recovery responses must not reveal account existence.
5. `429` is handled as an Auth rate-limit response where returned by the platform.
6. Secure client-IP forwarding must be used where required by the deployment architecture.
7. Q5 does not replace Q2 Active or Node 1 authorization.
8. Platform defaults are not immutable Freight application constants.

## 15. Final Locked Decision

```text
Q5 = 🔒 LOCKED

Supabase-native Auth rate limiting for MVP.
No custom Redis/Upstash limiter initially.
No hard account lockout.
Generic authentication/recovery responses.
Correct 429 handling.
Secure client-IP forwarding where required.
Future layered application controls require a new evidence-based decision.
```

## 16. Project Status

```text
Q1 = 🔒 LOCKED
Q2 = 🔒 LOCKED
Q3 = 🔒 LOCKED
Q4 = 🔒 LOCKED
Q5 = 🔒 LOCKED
Implementation = ⏸️ PAUSED
```

Q1–Q5 must not be reopened unless new evidence creates a genuine conflict.

## Next

**Q6 — RLS / service-role boundary.**

Follow the same workflow:

```text
Read authoritative records
        ↓
Investigate Q6
        ↓
Evidence
        ↓
Decision
        ↓
Independent review
        ↓
Ayush approval
        ↓
Q6 LOCK
        ↓
Continue to Q7
```

**No implementation is authorized by this Q5 lock.**