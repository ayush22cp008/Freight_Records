# Chat12 — Node 2 Claude Independent Review: Q3 Session Lifecycle / Refresh

Review only:

`03_IMPLEMENTATION/implementation_reports/Chat12_Node2_Report_Q3_Session_Lifecycle_Refresh.md`

## Context

Q1, Q2, and Q4 are already locked. Do not reopen them.

```text
Q1 = atomic Auth User → Freight Identity creation
Q2 = email confirmation required before normal authenticated onboarding and Active access
Q4 = exactly one Freight Identity per Auth User + DB UNIQUE(auth_user_id)
```

Q2 Active invariant:

```text
ACTIVE
= email_confirmed
AND verification_status = VERIFIED
AND trusted_role IS NOT NULL
```

Implementation remains paused.

## Review These Specific Issues

### 1. Middleware refresh

Verify whether the report is correct in recommending Next.js Middleware with `@supabase/ssr` for session refresh.

Do not blindly accept the statement that `getUser()` “automatically handles everything.” Verify the actual Supabase/SSR cookie-refresh behavior and explain the correct boundary.

### 2. Session vs authorization

Verify that the report correctly separates:

```text
Valid session = authenticated identity

Active + Node 1 authorization = business access
```

A valid session must not automatically grant Freight business access.

### 3. Q2 Active gate

The report must NOT use only the JWT `email_verified` claim as the authoritative Active decision.

Verify that protected access ultimately respects:

```text
email_confirmed
AND verification_status = VERIFIED
AND trusted_role IS NOT NULL
```

Also check whether stale JWT/session state could cause an unconfirmed verified user to retain access.

### 4. Layer handoff

Review this architecture:

```text
Request
 ↓
Middleware/session handling
 ↓
Server/API authentication
 ↓
Freight Identity lookup
 ↓
Q2 Active gate
 ↓
Node 1 authorization
 ↓
Business operation
```

Identify any missing or incorrect boundary.

### 5. Security

Check:

- access-token expiry;
- refresh-token rotation/replay;
- logout;
- invalid refresh token;
- stale claims;
- cookie security;
- CSRF considerations if relevant;
- email-change/reconfirmation;
- PENDING/REJECTED users with valid sessions.

## Required Verdict

Return exactly one:

```text
APPROVE
APPROVE WITH CONCERNS
NEEDS REVISION
BLOCKED
```

Then provide:

1. What is correct.
2. Any real concerns.
3. Exact corrections required.
4. Recommended final Q3 policy.
5. Required Node 2 contract changes.
6. Missing acceptance tests.
7. Confirm:

```text
Q1 = LOCKED
Q2 = LOCKED
Q4 = LOCKED
Implementation = NOT GRANTED
```

## Hard Constraints

```text
DO NOT IMPLEMENT.
DO NOT MODIFY CODE.
DO NOT MODIFY DATABASE MIGRATIONS.
DO NOT MODIFY SUPABASE CONFIGURATION.
DO NOT LOCK Q3.
DO NOT REOPEN Q1, Q2, OR Q4.
DO NOT TREAT THIS REVIEW AS AYUSH APPROVAL.
```

The purpose is to independently validate Q3 before Ayush makes the final decision.