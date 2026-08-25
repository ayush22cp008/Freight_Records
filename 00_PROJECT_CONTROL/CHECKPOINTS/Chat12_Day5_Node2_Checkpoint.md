# Chat12 — Day 5 Node 2 Continuation Checkpoint

**Project:** Freight — AI Builders Hackathon  
**Node:** Node 2 — Authentication + Identity  
**Chat:** Chat12  
**Checkpoint:** Day 5 continuation  
**Status:** ACTIVE / PARTIAL DECISIONS LOCKED / IMPLEMENTATION PAUSED

## 1. Purpose

This checkpoint is the continuation handoff for the next ChatGPT session. It records what was investigated and decided in Chat12 so work can resume without reconstructing this chat.

The authoritative workflow remains the uploaded `general-project-setup` skill and the Freight Records repository.

## 2. Current Node 2 roadmap questions

The Day 4 Node 2 checkpoint identified seven blocking decisions:

1. Signup / onboarding consistency
2. Email-confirmation policy
3. Session lifecycle / refresh
4. One-user → one-identity enforcement
5. Authentication rate limiting
6. RLS / service-role boundary
7. Final acceptance-test matrix

Authoritative source:

`00_PROJECT_CONTROL/CHECKPOINTS/Chat11_Day4_Node2_Checkpoint.md`

## 3. Decisions resolved in Chat12

### Q1 — Signup / onboarding consistency

**STATUS: DECIDED / LOCKED FOR NODE 2**

Selected direction:

```text
Supabase Auth User creation
        ↓
PostgreSQL transaction
        ↓
auth.users INSERT trigger
        ↓
Generic Freight Identity (PENDING)
        ↓
Atomic success / rollback
```

The trigger is the selected atomic identity-creation mechanism. Server-side compensation is not the primary consistency mechanism.

Claude independently reviewed Q1 and returned **APPROVE**.

Source:

`03_IMPLEMENTATION/implementation_reports/Chat12_Node2_Claude_Report_Review_Q1_Q4.md`

### Q4 — One-user → one-identity enforcement

**STATUS: DECIDED / LOCKED FOR NODE 2**

Authoritative invariant:

```text
1 Auth User
    ↓
exactly 1 Freight Identity
```

The database must enforce uniqueness of the Auth User reference in the Freight Identity table:

```text
freight_identities.auth_user_id = UNIQUE
```

Important distinction:

```text
PostgreSQL trigger
= atomic creation mechanism

UNIQUE(auth_user_id)
= database-level one-identity enforcement
```

The trigger alone is not sufficient as the uniqueness guarantee.

Claude reviewed Q4 and returned **CONCERN** only because the previous draft did not explicitly specify the database uniqueness constraint. The constraint is now explicitly locked as the Q4 decision. No Q4 blocker was identified.

Source:

`03_IMPLEMENTATION/implementation_reports/Chat12_Node2_Claude_Report_Review_Q1_Q4.md`

## 4. Identity / verification model established in Chat12

The leading and approved-for-further-design model is **Model C**:

```text
Auth User
   ↓
Generic Freight Identity
   ↓
requested_role = Driver / Company
verification_status = PENDING
trusted_role = NULL
   ↓
Evidence provided
   ↓
Ayush reviews evidence
   ↓
VERIFIED
   ↓
trusted_role = Driver / Company
```

### Requested role vs trusted role

`requested_role` is user-provided intent only.

It is NOT authorization.

The following are server-controlled:

```text
verification_status = PENDING | VERIFIED | REJECTED
trusted_role = NULL | Driver | Company
```

A client-supplied role, user metadata role, or requested role must never be treated as proof of authorization.

### Hackathon verification model

The selected MVP model is controlled human review.

```text
Driver  → Driving Licence evidence
Company → GST evidence/details
Verifier → Ayush
```

The user cannot approve their own evidence or directly change verification status/trusted role.

## 5. Security boundary established

A PENDING identity exists for consistency/onboarding but is not a trusted Driver or Company.

PENDING/REJECTED identities must not:

- perform protected Driver-only operations;
- perform protected Company-only operations;
- appear as eligible Drivers in marketplace queries;
- create/own protected business resources requiring a trusted role;
- bypass verification using client-controlled role fields.

## 6. Important distinction: decision lock vs implementation verification

The Q1/Q4 decisions are locked conceptually, but implementation is NOT complete or verified.

```text
Decision locked
≠
Code implemented
≠
Database constraint verified
≠
Trigger tested
```

Implementation remains paused until the remaining Node 2 blocking questions are resolved and the complete Node 2 contract is locked.

## 7. Current Node 2 contract

Primary architecture record:

`02_ARCHITECTURE/Chat11_Node2_Authentication_Identity_Contract_DRAFT.md`

It was updated in Chat12 with the Q1/Q4 decisions and the explicit database uniqueness requirement.

Current status remains DRAFT because the full Node 2 contract is not yet locked.

## 8. Investigations completed

### Pending identity vs verification

Investigation prompt:

`03_IMPLEMENTATION/prompts/Chat12_Node2_Investigation_Pending_Identity_vs_Verification.md`

Report:

`03_IMPLEMENTATION/implementation_reports/Chat12_Node2_Report_Pending_Identity_vs_Verification.md`

Conclusion: Model C is the leading model.

### Claude Q1/Q4 independent review

Prompt:

`03_IMPLEMENTATION/prompts/Chat12_Node2_Claude_Review_Q1_Q4.md`

Report:

`03_IMPLEMENTATION/implementation_reports/Chat12_Node2_Claude_Report_Review_Q1_Q4.md`

Conclusion:

```text
Q1 = APPROVE
Q4 = CONCERN → resolved by explicit DB UNIQUE(auth_user_id)
Overall = NEEDS REVISION at the time of Claude review
```

Ayush then approved the Q4 resolution.

## 9. Remaining Node 2 blocking questions

Do NOT skip these and do NOT invent replacements.

### Q2 — Email-confirmation policy

Need to decide:

- Is email confirmation required?
- What happens when email is unconfirmed?
- Can verification proceed before email confirmation?
- When does a verified identity become active/usable?
- What is the state ordering between:

```text
Identity exists
Email confirmed
Verification completed
Trusted role
Active/usable account
```

### Q3 — Session lifecycle / refresh

Need to define:

- session establishment;
- server-side verification;
- refresh behavior;
- logout;
- expired/invalid session behavior;
- cookie/token handling;
- protected-route behavior during refresh failure.

### Q5 — Authentication rate limiting

Need to define:

- per-IP controls;
- per-account/credential identifier controls;
- shared state where required;
- fail-open/fail-closed behavior if limiter state is unavailable;
- ordering relative to identity lookup;
- failure response behavior.

### Q6 — RLS / service-role boundary

Need to define the exact Node 2 boundary for:

- authenticated client access;
- RLS;
- server-side/service-role operations;
- identity/verification operations;
- preventing service-role access from becoming a substitute for authorization.

### Q7 — Final acceptance-test matrix

Need to convert the Node 2 contract into a final testable matrix covering at minimum:

- one-user/one-identity;
- trusted role establishment;
- verification boundaries;
- signup atomicity;
- authentication;
- session lifecycle;
- role enforcement;
- service-role/RLS boundary;
- rate limiting;
- identity handoff.

## 10. Immediate next action

**NEXT: Q2 — Email-confirmation policy.**

Do not implement Node 2 yet.

Recommended workflow for Q2:

```text
Read current Node 2 contract + Day 4 checkpoint
        ↓
Investigate Q2 properly
        ↓
Evidence
        ↓
Decision
        ↓
Record decision
        ↓
Independent Claude review where appropriate
        ↓
Ayush approval
        ↓
Move to Q3
```

Do not jump directly from a symptom to implementation.

## 11. Current project status

```text
Node 2                     = ACTIVE
Q1 Signup consistency      = LOCKED DECISION
Q4 One-user/one-identity   = LOCKED DECISION
Model C                    = SELECTED DIRECTION
Manual verification        = SELECTED HACKATHON MVP DIRECTION
Q2 Email confirmation     = OPEN / NEXT
Q3 Session lifecycle      = OPEN
Q5 Rate limiting          = OPEN
Q6 RLS/service-role       = OPEN
Q7 Acceptance matrix      = OPEN
Full Node 2 contract      = NOT LOCKED
Implementation            = PAUSED
```

## 12. Continuation instruction for next ChatGPT session

Start by reading:

```text
00_PROJECT_CONTROL/ROADMAP.md
00_PROJECT_CONTROL/CURRENT_STATUS.md
00_PROJECT_CONTROL/PROJECT_STATE.md
00_PROJECT_CONTROL/CHECKPOINTS/Chat11_Day4_Node2_Checkpoint.md
00_PROJECT_CONTROL/CHECKPOINTS/Chat12_Day5_Node2_Checkpoint.md
02_ARCHITECTURE/Chat11_Node2_Authentication_Identity_Contract_DRAFT.md
03_IMPLEMENTATION/implementation_reports/Chat12_Node2_Claude_Report_Review_Q1_Q4.md
```

Then continue with **Q2 — Email-confirmation policy**.

Do not re-open Q1/Q4 unless new evidence creates a genuine conflict with the Records.
