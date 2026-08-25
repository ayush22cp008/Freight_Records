# Chat12 — Node 2 Investigation: Q5 Authentication Rate Limiting

**Project:** Freight — AI Builders Hackathon  
**Node:** Node 2 — Authentication + Identity  
**Question:** Q5 — Authentication rate limiting  
**Status:** INVESTIGATION ONLY / NO IMPLEMENTATION AUTHORIZATION

## Objective

Determine the correct authentication rate-limiting policy for Freight before Node 2 implementation resumes.

Investigate current repository architecture, current Supabase/Auth behavior, relevant platform documentation, security best practices, and realistic MVP constraints. Produce a decision-ready report. Do not implement anything.

## Locked Constraints

Do NOT reopen:

```text
Q1 = LOCKED — atomic Auth User → Freight Identity creation
Q2 = LOCKED — email confirmation before normal authenticated onboarding and Active access
Q3 = LOCKED — Middleware-centered session refresh + live Freight DB Active gate
Q4 = LOCKED — exactly one Freight Identity per Auth User + DB UNIQUE(auth_user_id)
```

Q2 Active invariant remains:

```text
ACTIVE
= email_confirmed
AND verification_status = VERIFIED
AND trusted_role IS NOT NULL
```

Q3 authentication invariant remains:

```text
Valid Session
    ≠ Active Freight Account
    ≠ Authorized for every operation
```

Implementation remains paused.

## Records to Read First

Read the current authoritative project-control and architecture records before investigating:

```text
00_PROJECT_CONTROL/ROADMAP.md
00_PROJECT_CONTROL/CURRENT_STATUS.md
00_PROJECT_CONTROL/PROJECT_STATE.md
00_PROJECT_CONTROL/CHECKPOINTS/Chat12_Day5_Node2_Checkpoint.md
02_ARCHITECTURE/Chat11_Node2_Authentication_Identity_Contract_DRAFT.md
03_IMPLEMENTATION/implementation_reports/Chat12_Node2_Report_Email_Confirmation_Policy.md
03_IMPLEMENTATION/implementation_reports/Chat12_Node2_Report_Q3_Session_Lifecycle_Refresh.md
03_IMPLEMENTATION/implementation_reports/Chat12_Node2_Report_Q3_Session_Lifecycle_Refresh_Claude_approved.md
```

Also inspect the actual authentication-related source/configuration where accessible, without modifying it.

Use current official Supabase documentation for platform-specific behavior. Do not assume a generic rate limiter is already provided by Supabase unless verified.

## Core Problem

Determine how Freight should limit abusive authentication traffic while preserving legitimate users.

The investigation must cover at least:

```text
Login / sign-in
Signup
Password recovery / reset requests
Email confirmation / resend flows
Other authentication endpoints actually present in the application
```

Do not automatically apply the same limit to every endpoint. Identify which endpoints need which controls.

## Questions to Resolve

### 1. What attacks are we defending against?

Analyze:

- password brute force;
- credential stuffing;
- repeated signup abuse;
- password-reset email abuse;
- confirmation-email resend abuse;
- account enumeration;
- IP-based distributed attacks;
- resource exhaustion against authentication endpoints;
- abuse of one victim account by many IP addresses.

Separate threats that rate limiting can materially mitigate from threats requiring other controls.

### 2. Which dimensions should be rate-limited?

Compare:

```text
Per IP address
Per account / credential identifier
Per endpoint
Per device/session identifier, if justified
Combined / layered limits
```

Determine whether Freight needs a layered policy such as:

```text
IP limit
    +
account/identifier limit
    +
endpoint-specific limit
```

Explain why.

Do not assume the identifier is always a confirmed account. Consider unknown/unregistered identifiers without creating an account-enumeration oracle.

### 3. What should be limited?

Define appropriate controls for:

```text
Password login attempts
Signup attempts
Password-reset initiation
Confirmation-email resend
Other auth operations
```

Distinguish between:

```text
attempt rate
email-send rate
successful authentication rate
failure rate
```

### 4. What are the time windows and thresholds?

Do NOT invent arbitrary numbers.

Research reasonable security/MVP ranges and recommend concrete thresholds only if evidence supports them.

If exact thresholds should remain configurable rather than locked in architecture, say so.

Consider:

- burst limits;
- short windows;
- longer windows;
- progressive backoff;
- account lockout risks;
- legitimate shared networks such as colleges, offices, NAT gateways, and mobile carriers.

### 5. Where does limiter state live?

Compare:

```text
In-memory process state
Database table
Redis / managed shared store
Platform-provided rate limiting
Other shared durable state
```

Determine whether a multi-instance deployment makes per-process memory unsafe or insufficient.

The policy must explicitly distinguish:

```text
single-process MVP assumption
vs
multi-instance-safe architecture
```

Do not add a new infrastructure dependency without explaining why it is necessary for the hackathon MVP.

### 6. What happens when the limiter is unavailable?

Define fail behavior:

```text
Fail-open
    = allow authentication request if limiter cannot be checked

Fail-closed
    = deny/block authentication request if limiter cannot be checked
```

Evaluate the security and availability consequences for each authentication endpoint.

If different endpoints need different behavior, state that explicitly.

### 7. Ordering relative to identity lookup

Determine the safe request order.

For example:

```text
Request
 ↓
Rate limit
 ↓
Authentication provider
 ↓
Freight identity lookup
 ↓
Active/authorization checks
```

or another evidence-supported order.

Pay special attention to whether looking up an account before rate limiting can create an enumeration or resource-exhaustion problem.

Do not expose whether an email/account exists through rate-limit responses.

### 8. Failure responses

Define expected behavior when limits are exceeded.

Evaluate:

```text
HTTP 429
Retry-After
Generic authentication error
Login cooldown
Password-reset cooldown
```

Ensure responses do not reveal whether an account exists.

Define the difference between:

```text
Authentication failure
Rate-limit rejection
Account state rejection
Authorization rejection
```

### 9. Interaction with Q2 and Q3

Verify that rate limiting does not bypass or alter the locked policies.

Q2:

```text
Email confirmation
    ↓
normal authenticated onboarding
    ↓
Active gate
```

Q3:

```text
Middleware session refresh
    ↓
valid session
    ↓
live Freight DB Active gate
```

Rate limiting is an abuse-control layer, not an authorization mechanism.

A rate-limited user must not receive protected access merely because a request was allowed.

A non-rate-limited request must still pass Q2 Active and Node 1 authorization.

### 10. Account lockout / denial-of-service risk

Investigate whether per-account lockout is appropriate.

Example:

```text
Attacker knows victim@example.com
        ↓
Sends many failed attempts
        ↓
Victim gets locked out
```

Determine whether progressive delay/rate limiting is safer than hard account lockout for the MVP.

### 11. Distributed attacks

Analyze:

```text
One IP → many accounts
Many IPs → one account
Many IPs → many accounts
```

Determine which combinations the recommended policy can detect and which require additional infrastructure or provider-side controls.

### 12. Privacy / enumeration

Ensure the rate limiter does not create a side channel such as:

```text
Known account → different response
Unknown account → different response
```

Recommend generic responses where appropriate.

### 13. Supabase-specific controls

Verify current official Supabase capabilities relevant to:

- Auth endpoint rate limits;
- abuse prevention;
- email sending limits;
- password recovery limits;
- CAPTCHA or bot protection if relevant;
- configuration limits;
- whether project-level limits can be relied upon as the complete Freight policy.

Clearly separate:

```text
Supabase platform protection
vs
Freight application-level rate limiting
```

Do not duplicate a platform control unnecessarily.

## Candidate Policies

Compare at least these candidates:

### Policy A — IP-only rate limiting

Limit authentication requests by source IP.

### Policy B — Layered IP + account/identifier + endpoint controls

Use multiple dimensions with endpoint-specific thresholds.

### Policy C — Platform-only controls

Rely primarily on Supabase's built-in authentication/email protections.

### Policy D — Hard account lockout

Lock the account after repeated failed authentication attempts.

You may recommend a hybrid if evidence supports it.

## Required State / Behavior Matrix

Complete this matrix:

| Endpoint | Scenario | Limiter State | Authentication Result | Response | Account Existence Leaked? |
|---|---|---|---|---|---|
| Login | normal valid attempt | under limit | provider handles | ? | ? |
| Login | repeated failures | limit exceeded | blocked | ? | ? |
| Login | many IPs targeting one account | ? | ? | ? | ? |
| Signup | repeated attempts | ? | ? | ? | ? |
| Password reset | normal request | ? | ? | ? | ? |
| Password reset | repeated request | ? | ? | ? | ? |
| Email resend | repeated request | ? | ? | ? | ? |
| Any auth endpoint | limiter unavailable | ? | ? | ? | ? |

## Required Architecture

Describe the recommended flow:

```text
Client
 ↓
Authentication endpoint
 ↓
Rate-limit decision
 ↓
Supabase Auth operation
 ↓
Freight identity handling where required
 ↓
Q2/Q3 authentication state
 ↓
Node 1 authorization where required
```

Clearly identify which layers are responsible for:

```text
Abuse control
Authentication
Identity state
Active state
Authorization
```

## Required Security Analysis

Explicitly analyze:

- brute force;
- credential stuffing;
- distributed login attempts;
- account lockout DoS;
- password-reset email bombing;
- confirmation email bombing;
- enumeration;
- spoofed/rotated IPs;
- shared NAT/public IPs;
- limiter state tampering;
- fail-open behavior;
- fail-closed behavior;
- race conditions in counters;
- concurrent requests crossing a threshold;
- clock/window consistency;
- abuse of public signup endpoints.

## Required Decision

Produce a recommendation for:

```text
Q5 = <AUTHENTICATION RATE-LIMITING POLICY>
```

The recommendation must explicitly define:

1. protected authentication endpoints;
2. rate-limit dimensions;
3. endpoint-specific controls;
4. threshold strategy;
5. shared-state requirement;
6. fail-open/fail-closed policy;
7. request ordering;
8. failure responses;
9. enumeration protection;
10. account-lockout policy;
11. Supabase vs Freight responsibility;
12. observability/abuse logging requirements.

## Required Output

Return a structured report containing:

### 1. Executive conclusion

State the recommended Q5 policy clearly.

### 2. Evidence

Separate:

```text
Verified repository evidence
Verified Supabase/platform evidence
Security best-practice evidence
Inference
Unknowns
```

### 3. Threat model

Summarize the realistic attacks relevant to Freight's MVP.

### 4. Candidate comparison

Compare Policies A/B/C/D or evidence-justified alternatives.

### 5. Recommended architecture

Show the complete rate-limiting flow.

### 6. Threshold policy

Provide concrete values only where justified; otherwise define configurable ranges/defaults.

### 7. State/behavior matrix

Complete the required matrix.

### 8. Security analysis

Explain the major risks and how the recommended policy handles them.

### 9. Q2/Q3 interaction

Explicitly explain how rate limiting interacts with:

```text
email confirmation
session refresh
Active status
Node 1 authorization
```

### 10. Contract changes required

List exact Node 2 contract changes. Do not modify the contract during investigation.

### 11. Acceptance tests

Include tests for:

- IP burst limit;
- per-account/identifier limit;
- distributed attempts;
- signup abuse;
- password-reset abuse;
- confirmation-resend abuse;
- 429 behavior;
- Retry-After where appropriate;
- no account enumeration;
- limiter unavailable;
- concurrent requests near threshold;
- legitimate shared-IP users;
- Q2 Active gate still enforced;
- Q3 session lifecycle still enforced;
- Node 1 authorization still enforced.

### 12. Remaining unknowns

List only genuine blockers.

### 13. Implementation status

```text
Implementation authorization = NOT GRANTED
```

## Hard Constraints

```text
DO NOT IMPLEMENT.
DO NOT MODIFY APPLICATION CODE.
DO NOT MODIFY DATABASE MIGRATIONS.
DO NOT MODIFY SUPABASE AUTH CONFIGURATION.
DO NOT MODIFY THE NODE 2 CONTRACT.
DO NOT LOCK Q5.
DO NOT REOPEN Q1.
DO NOT REOPEN Q2.
DO NOT REOPEN Q3.
DO NOT REOPEN Q4.
```

## Completion Gate

The investigation is complete only when:

```text
[ ] Project-control records read
[ ] Q1/Q2/Q3/Q4 constraints preserved
[ ] Current Node 2 contract read
[ ] Current auth source/config inspected where available
[ ] Supabase rate-limit behavior verified
[ ] Authentication threats identified
[ ] Protected auth endpoints identified
[ ] Rate-limit dimensions compared
[ ] Threshold strategy defined
[ ] Shared-state requirement analyzed
[ ] Fail-open/fail-closed behavior decided
[ ] Request ordering analyzed
[ ] Failure responses defined
[ ] Enumeration protection analyzed
[ ] Account-lockout policy analyzed
[ ] Supabase vs Freight responsibilities separated
[ ] State matrix completed
[ ] Candidate policies compared
[ ] Recommended Q5 policy stated
[ ] Acceptance tests proposed
[ ] Contract changes identified
[ ] Implementation remains paused
```

After the investigation:

```text
Q5 Investigation Report
        ↓
Independent Claude Review
        ↓
Ayush decision
        ↓
Q5 Lock
        ↓
Move to Q6
```

Do not skip the independent-review and Ayush-approval gates.