# Chat12 — Node 2 Decision Report: Q3 Session Lifecycle & Refresh

## 1. Final Policy Direction

**Q3 = Policy B — Middleware-centered session refresh**, with the security refinements approved during the Q3 decision discussion.

This report is now the refined Q3 decision candidate. Q3 is **not locked yet**; it requires the final independent review and Ayush approval gate.

The policy is:

```text
User Login
    ↓
Supabase Session
    ↓
Scoped Next.js Middleware
    ↓
supabase.auth.getUser()
    ↓
Session validation / refresh when required
    ↓
Protected request
    ↓
Live Freight Identity DB lookup
    ↓
Q2 Active Gate
    ↓
Node 1 Authorization
    ↓
Business Operation
```

## 2. Locked Constraints

Do not reopen:

```text
Q1 = LOCKED — atomic Auth User → Freight Identity creation
Q2 = LOCKED — email confirmation + VERIFIED + trusted_role required for Active access
Q4 = LOCKED — exactly one Freight Identity per Auth User + DB UNIQUE(auth_user_id)
```

Q2 Active invariant:

```text
ACTIVE
= email_confirmed
AND verification_status = VERIFIED
AND trusted_role IS NOT NULL
```

## 3. Five Q3 Decisions Approved by Ayush

### Decision 1 — Active status source

The Freight Active state is authoritative in the Freight Identity database.

Every protected business request must use current DB state to evaluate:

```text
email_confirmed
verification_status
trusted_role
```

Active status must not be cached as an authorization decision in a cookie, JWT claim, or middleware/session object.

### Decision 2 — JWT role

JWT/session state identifies/authenticates the user but is not the final Freight Active/authorization decision.

In particular:

```text
JWT email_verified
    ≠
Freight Active
```

Supabase email-confirmation information may inform authentication state, but protected business access must still use the live Freight Identity state and Node 1 authorization.

### Decision 3 — CSRF

For cookie-authenticated state-changing requests, the MVP policy is:

```text
SameSite cookie protection
+
Origin validation
+
normal authentication/authorization checks
```

A separate CSRF-token framework is not required for the MVP unless later implementation evidence shows that the actual route architecture requires it.

### Decision 4 — PENDING / REJECTED sessions

PENDING or REJECTED users may retain a valid authentication session, but they immediately lose protected business access through the live DB Active gate.

```text
Valid session
    ↓
DB says PENDING/REJECTED
    ↓
403 / protected access denied
```

Forced session revocation is not the normal authorization mechanism. It may be used later for explicit security events if required.

### Decision 5 — Middleware scope

Middleware is scoped to the authenticated/protected application surface.

Public pages and static assets should bypass authentication refresh/validation where they do not require it.

The exact matcher route list is an implementation deliverable and must be finalized from the actual application route structure before implementation.

## 4. Session vs Authorization

The architecture must preserve this separation:

```text
Valid Session
    = authenticated identity

Freight Active Gate
    = current account/verification state

Node 1 Authorization
    = permission for the requested business operation
```

Therefore:

```text
Valid Session
    ≠
Active Freight Account
    ≠
Authorized for every operation
```

A confirmed but PENDING user may have a valid session but no protected Driver/Company business access.

A VERIFIED user whose email becomes unconfirmed may retain an authentication session but must immediately lose protected business access.

## 5. Session Lifecycle

```text
Login
 ↓
Supabase Auth establishes session
 ↓
Session credentials are maintained through the supported @supabase/ssr cookie mechanism
 ↓
Protected request
 ↓
Scoped Middleware validates current user with getUser()
 ↓
If refresh is required and valid refresh credentials exist
 ↓
Updated session cookies are written to the response
 ↓
Server/API handles protected operation
 ↓
Live Freight Identity lookup
 ↓
Q2 Active Gate
 ↓
Node 1 Authorization
 ↓
Business operation
```

Do not interpret `getUser()` as an unconditional guarantee that every refresh succeeds. Refresh success/failure depends on the current session and refresh-token state.

## 6. Refresh Failure / Invalid Session

If the session cannot be validated or refreshed:

```text
Invalid/expired session
    ↓
Protected browser request → redirect/login handling
Protected API request     → 401
```

Session cleanup should remove invalid local session cookies where the framework's supported flow requires it.

## 7. Cookie / Token Boundary

Use the supported `@supabase/ssr` server-client cookie mechanism.

Session credentials must use secure cookie attributes appropriate to the deployed environment, including:

```text
HttpOnly
Secure (production)
SameSite=Lax or stricter where compatible
```

Do not expose refresh credentials to application JavaScript unnecessarily.

## 8. Active Gate Must Use Live Freight DB State

This is a mandatory security invariant:

```text
Every protected business request
        ↓
Live Freight Identity DB state
        ↓
email_confirmed
verification_status
trusted_role
        ↓
Active / Not Active
```

The following must **never** be treated as the authoritative Active decision:

```text
JWT email_verified alone
JWT custom verification claims
cached middleware state
cached client state
old session metadata
```

This prevents a still-valid old access token from retaining business access after an administrator changes the user's Freight state.

## 9. Q2 Interaction

Q2 remains authoritative for account activation:

```text
Active
= email_confirmed
AND verification_status = VERIFIED
AND trusted_role IS NOT NULL
```

Example:

```text
ACTIVE user
    ↓
verification_status changed to REJECTED
    ↓
next protected request
    ↓
DB lookup returns REJECTED
    ↓
403
```

The same principle applies if email confirmation becomes false after an email change.

## 10. CSRF Policy

Because browser authentication uses cookies, state-changing routes must enforce the approved MVP boundary:

```text
SameSite cookie protection
+
Origin validation
+
valid authentication session
+
Q2 Active gate
+
Node 1 authorization
```

The exact implementation mechanism is deferred to implementation work after the Node 2 contract is locked.

If the actual route architecture introduces cross-origin authenticated requests, revisit this decision with evidence before implementation.

## 11. Middleware Scope

The final implementation must define an explicit matcher for the authenticated/protected route surface.

Requirements:

- protected routes must not bypass session refresh/validation;
- public routes may bypass authentication processing;
- static assets and framework internals should not incur unnecessary auth processing;
- API routes requiring authentication must remain covered.

The concrete matcher belongs to implementation design and must be derived from the actual route tree.

## 12. PENDING / REJECTED Session Policy

Normal policy:

```text
PENDING / REJECTED
    ↓
Authentication session may remain valid
    ↓
Live Active gate denies protected business access
```

The project does not require forced logout merely because verification state changes.

Explicit session revocation may be introduced for separate security events, but that is not the normal Node 2 authorization mechanism.

## 13. State Matrix

| Session | Email | Verification | Trusted Role | Active | Protected Access | Expected Behavior |
|---|---|---|---|---|---|---|
| None | Confirmed | VERIFIED | Driver/Company | YES | NO | 401 / login handling |
| Valid | Confirmed | PENDING | NULL | NO | NO | Session valid; protected operation returns 403 |
| Valid | Confirmed | VERIFIED | Driver/Company | YES | YES | Continue to Node 1 authorization |
| Valid | Unconfirmed | VERIFIED | Driver/Company | NO | NO | Session may remain valid; protected operation returns 403 |
| Expired | Confirmed | VERIFIED | Driver/Company | YES | YES* | Middleware attempts supported refresh; if successful continue |
| Invalid | Confirmed | VERIFIED | Driver/Company | YES | NO | Session rejected; protected request returns 401 / login handling |
| Valid | Confirmed | REJECTED | NULL | NO | NO | Session may remain valid; protected operation returns 403 |

## 14. Candidate Policy Decision

### Policy A — Server-only session verification

Rejected for the primary Next.js App Router refresh boundary because Server Components cannot directly write refreshed cookies to the response.

### Policy B — Middleware-centered refresh

**SELECTED.**

Middleware with the supported `@supabase/ssr` server-client pattern provides the appropriate session-refresh boundary, while protected server operations independently enforce the live Freight Active gate and Node 1 authorization.

### Policy C — Client-only session management

Rejected. Client-side session state must not be the authoritative protected-server authentication or authorization boundary.

## 15. Security Analysis

### Access-token theft

Short-lived access tokens reduce exposure, but token possession remains a risk. Protected operations still require current session validation and the live Freight Active gate.

### Refresh-token replay

Use Supabase's supported refresh-token rotation behavior. Do not build a custom refresh-token mechanism.

### Stale claims

JWT claims are not the authoritative Freight Active state. Live DB state prevents a stale token from preserving business access after a verification/email-state change.

### Browser token exposure

Use the supported secure cookie boundary and avoid exposing refresh credentials to application JavaScript unnecessarily.

### Logout

Normal logout should use the supported Supabase sign-out flow and clear the local session cookie state.

### Email change

If email confirmation becomes false, the user may retain authentication temporarily, but the live Q2 Active gate immediately denies protected business access until confirmation is restored.

## 16. Acceptance Tests

1. Active user with valid session can reach a protected route after Node 1 authorization.
2. Confirmed PENDING user can authenticate but receives 403 for protected Driver/Company operations.
3. Rejected user with valid session receives 403 for protected business operations.
4. Expired access token with valid refresh state is refreshed through the supported Middleware flow.
5. Invalid refresh/session state produces 401/login handling rather than business access.
6. Logout removes the usable session and subsequent protected requests fail authentication.
7. Active user whose `verification_status` changes to REJECTED while the access token remains valid receives 403 on the next protected request.
8. Active user whose email becomes unconfirmed receives 403 on the next protected request even if an old session/token remains valid.
9. JWT `email_verified` alone cannot grant protected business access.
10. `verification_status` and `trusted_role` are read from current Freight Identity state for protected requests.
11. State-changing authenticated requests enforce the approved SameSite + Origin validation boundary.
12. Public/static routes do not unnecessarily run protected-route session processing.
13. Every protected API route remains covered by the final Middleware matcher.
14. Node 1 authorization still runs after authentication and Q2 Active checks.

## 17. Contract Changes Required

Before implementation, the Node 2 contract should state:

1. Middleware is the session refresh boundary for the authenticated/protected application surface.
2. `getUser()`/server-side authentication verification is used rather than treating local JWT decoding as sufficient verification.
3. Session validity is separate from Active account state.
4. Active status is derived from live Freight Identity DB state.
5. JWT claims are not the authoritative Freight authorization state.
6. Cookie security and SameSite/Origin protections are required for the browser session boundary.
7. PENDING/REJECTED sessions may remain authenticated but cannot access protected business operations.
8. The exact Middleware matcher must cover all protected routes.
9. Node 1 authorization remains the final operation-level authorization boundary.

Do not implement these changes until Q3 is formally locked.

## 18. Independent Review Follow-up

Claude's first review returned:

```text
APPROVE WITH CONCERNS
```

The concerns were:

- live DB Active-gate requirement needed to be explicit;
- JWT email verification needed to be separated from Freight verification state;
- stale-state behavior needed an explicit test;
- CSRF needed an explicit policy;
- Middleware matcher scope needed to be an explicit deliverable;
- PENDING/REJECTED session handling needed an explicit decision.

All six points have now been addressed in this refined report based on Ayush's decisions.

A final independent review is still required.

## 19. Final Status

```text
Q1 = LOCKED
Q2 = LOCKED
Q3 = NOT LOCKED — refined and ready for final independent review
Q4 = LOCKED
Implementation authorization = NOT GRANTED
```

Next workflow:

```text
Refined Q3 report
    ↓
Final Claude review
    ↓
Ayush approval
    ↓
Q3 LOCK
    ↓
Next Node 2 question
```
