# Chat12 — Node 2 Investigation: Q3 Session Lifecycle / Refresh

**Project:** Freight — AI Builders Hackathon  
**Node:** Node 2 — Authentication + Identity  
**Question:** Q3 — Session lifecycle / refresh  
**Status:** INVESTIGATION ONLY / NO IMPLEMENTATION AUTHORIZATION

## Objective

Determine the correct authentication-session lifecycle for Freight before Node 2 implementation resumes.

Investigate evidence, recommend a policy, and define acceptance tests. Do not modify application code, database migrations, Supabase configuration, or the Node 2 contract.

## Locked Constraints

Do NOT reopen:

```text
Q1 = LOCKED — PostgreSQL-trigger-based atomic Auth User → Freight Identity creation
Q2 = LOCKED — email confirmation before normal authenticated onboarding and Active access
Q4 = LOCKED — exactly one Freight Identity per Auth User with DB UNIQUE(auth_user_id)
```

Q2 Active invariant:

```text
ACTIVE
= email_confirmed
AND verification_status = VERIFIED
AND trusted_role IS NOT NULL
```

A valid authentication session alone must never grant business authorization.

## Records to Read First

Read the current authoritative project-control and architecture records:

```text
00_PROJECT_CONTROL/ROADMAP.md
00_PROJECT_CONTROL/CURRENT_STATUS.md
00_PROJECT_CONTROL/PROJECT_STATE.md
00_PROJECT_CONTROL/CHECKPOINTS/Chat12_Day5_Node2_Checkpoint.md
00_PROJECT_CONTROL/CHECKPOINTS/Chat12_Day5_Node2_Q2_Lock_Checkpoint.md
02_ARCHITECTURE/Chat11_Node2_Authentication_Identity_Contract_DRAFT.md
03_IMPLEMENTATION/implementation_reports/Chat12_Node2_Report_Email_Confirmation_Policy.md
03_IMPLEMENTATION/implementation_reports/Chat12_Node2_Claude_Report_Review_Q2_Email_Confirmation_Policy.md
03_IMPLEMENTATION/implementation_reports/Chat12_Node2_Claude_Report_Review_Q1_Q4.md
```

Also inspect relevant current source code if accessible and use current authoritative Supabase documentation for platform behavior.

## Questions to Resolve

### 1. Session establishment

Determine exactly what happens after successful login:

```text
Credentials
 ↓
Supabase Auth
 ↓
Session / access token / refresh token
 ↓
Authenticated request
```

Identify what is created, where it is stored, and how the server validates it.

### 2. Access-token expiration

Determine:

- access-token lifetime relevant to the current project;
- what happens when it expires;
- whether the client automatically refreshes;
- how refresh interacts with server-rendered/protected requests;
- what happens if refresh fails.

Do not assume defaults; verify current Supabase behavior.

### 3. Refresh-token lifecycle

Determine:

- how refresh tokens are issued and used;
- whether refresh-token rotation applies;
- how refresh-token reuse/replay is handled;
- what invalidates a refresh token/session;
- how logout affects refresh capability.

### 4. Server-side authentication

Determine the recommended server-side boundary for protected requests:

```text
Request
 ↓
Authenticated Supabase session
 ↓
Server verifies current user
 ↓
Freight Identity lookup
 ↓
Q2 Active gate
 ↓
Node 1 authorization
 ↓
Business operation
```

Determine whether middleware is required, recommended, unnecessary, or only useful for selected routes.

Do not choose middleware merely because it is convenient; base the recommendation on actual framework/Supabase behavior.

### 5. Cookies / browser storage

Determine the safest session-credential handling for this architecture, including:

- cookies;
- HttpOnly/Secure/SameSite considerations;
- browser storage risks;
- server-side access to session state;
- exposure of access/refresh tokens to client JavaScript.

Keep implementation-specific details separate from policy decisions.

### 6. Logout

Define expected behavior for:

- normal logout;
- session invalidation;
- refresh-token invalidation;
- browser cookie cleanup;
- subsequent protected requests.

### 7. Expired / invalid sessions

Define exact policy for:

```text
No session
Expired access token
Invalid access token
Expired/invalid refresh token
Revoked session
Malformed authentication state
```

Determine whether each should:

```text
→ refresh
→ return 401
→ redirect to login
→ clear session
```

Separate API behavior from browser UI behavior where necessary.

### 8. Interaction with Q2

Verify that session validity and account activation remain separate:

```text
Valid session
    ≠
Active Freight account
```

A confirmed but PENDING user may have a valid authenticated session while still having no protected Driver/Company access.

A VERIFIED user whose email becomes unconfirmed must not retain protected business access merely because an old session/token exists.

Determine what session freshness is required to safely enforce this rule.

### 9. Email-change/session freshness edge case

Investigate:

```text
ACTIVE user
 ↓
email changes / reconfirmation required
 ↓
email_confirmed becomes false
```

Determine whether existing sessions/tokens can remain valid and whether their claims can become stale.

Recommend the policy for protected access during this state.

Do not prematurely select a specific RLS/JWT implementation; that belongs to Q6 where appropriate.

### 10. Session revocation / security events

Determine expected behavior after important security events, including where supported:

- password reset;
- password change;
- email change;
- email confirmation state change;
- explicit logout;
- account rejection/deactivation;
- suspicious session/reuse event.

Identify which events require immediate access loss versus eventual token expiry/refresh behavior.

## Required State Matrix

Complete this matrix:

| Session | Email | Verification | Trusted Role | Active | Protected Access | Expected Behavior |
|---|---|---|---|---|---|---|
| None | Confirmed | VERIFIED | Driver/Company | YES | ? | ? |
| Valid | Confirmed | PENDING | NULL | NO | ? | ? |
| Valid | Confirmed | VERIFIED | Driver/Company | YES | ? | ? |
| Valid | Unconfirmed | VERIFIED | Driver/Company | NO | ? | ? |
| Expired | Confirmed | VERIFIED | Driver/Company | YES | ? | ? |
| Invalid | Confirmed | VERIFIED | Driver/Company | YES | ? | ? |
| Valid | Confirmed | REJECTED | NULL | NO | ? | ? |

## Candidate Session Policies

Compare at least:

### Policy A — Server-verified session on protected requests + normal refresh

The application verifies the current authenticated session on protected requests and relies on the supported Supabase refresh lifecycle.

### Policy B — Middleware-centered session refresh

Use middleware as the primary mechanism for refreshing/maintaining session state before protected requests.

### Policy C — Client-only session management

Let the browser/client manage the session while protected server endpoints trust client-provided authentication state.

Treat Policy C as a security baseline to reject or accept based on evidence; do not assume it is acceptable.

You may recommend a hybrid if evidence supports it.

## Security Analysis

Explicitly investigate:

- stolen access tokens;
- stolen refresh tokens;
- refresh-token replay/reuse;
- stale JWT claims;
- session fixation;
- logout not actually invalidating access;
- browser token exposure;
- CSRF where cookie-based authentication is used;
- session confusion between users;
- protected-route access after account deactivation/email unconfirmation;
- service-role misuse as a substitute for authentication/authorization.

Do not conflate authentication with Node 1 authorization.

## Required Decision

Produce a recommendation for:

```text
Q3 = <SESSION LIFECYCLE POLICY>
```

The recommendation must define:

1. session establishment;
2. access-token validation;
3. refresh behavior;
4. refresh failure behavior;
5. logout;
6. expired/invalid session behavior;
7. credential storage boundary;
8. server-side request authentication;
9. interaction with Q2 Active status;
10. session freshness expectations.

## Required Output

Return a structured report containing:

### 1. Executive conclusion

State the recommended Q3 policy clearly.

### 2. Evidence

Separate:

```text
Verified repository evidence
Verified Supabase/platform evidence
Inference
Unknowns
```

### 3. Session lifecycle diagram

Show the complete recommended flow from login through refresh, logout, and failure.

### 4. State matrix

Complete the matrix above.

### 5. Candidate comparison

Compare Policies A/B/C or evidence-justified alternatives.

### 6. Security analysis

Explain the major threats and how the recommended lifecycle handles them.

### 7. Q2 interaction

Explicitly explain how session validity interacts with:

```text
email_confirmed
verification_status
trusted_role
Active
```

### 8. Recommended Q3 decision

State the exact policy and invariants.

### 9. Contract changes required

List exact changes needed in the Node 2 contract. Do not make those changes in this investigation.

### 10. Acceptance tests

Include tests for:

- login;
- valid protected request;
- expired access token;
- successful refresh;
- failed refresh;
- logout;
- invalid session;
- PENDING user with valid session;
- VERIFIED + confirmed user;
- VERIFIED + unconfirmed user;
- email change and stale-session behavior;
- rejected user;
- role/identity handoff to Node 1.

### 11. Remaining unknowns

Only genuine blockers.

### 12. Implementation status

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
DO NOT REOPEN Q1.
DO NOT REOPEN Q2.
DO NOT REOPEN Q4.
DO NOT LOCK Q3.
```

## Completion Gate

The investigation is complete only when:

```text
[ ] Project-control records read
[ ] Q1/Q2/Q4 constraints preserved
[ ] Current Node 2 contract read
[ ] Current source evidence inspected where available
[ ] Supabase session behavior verified
[ ] Session lifecycle defined
[ ] Refresh lifecycle defined
[ ] Logout defined
[ ] Expired/invalid behavior defined
[ ] Cookie/token boundary analyzed
[ ] Q2 Active interaction analyzed
[ ] Candidate policies compared
[ ] Security risks analyzed
[ ] State matrix completed
[ ] Recommended Q3 policy stated
[ ] Acceptance tests proposed
[ ] Contract changes identified
[ ] Implementation remains paused
```

After the investigation:

```text
Q3 Investigation Report
        ↓
Independent Claude Review
        ↓
Ayush approval / rejection
        ↓
Q3 Lock
        ↓
Move to next Node 2 question
```

Do not skip the independent-review and Ayush-approval gates.