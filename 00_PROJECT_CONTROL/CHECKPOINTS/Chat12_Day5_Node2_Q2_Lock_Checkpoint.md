# Chat12 — Day 5 Node 2 Q2 Lock Checkpoint

**Project:** Freight — AI Builders Hackathon  
**Node:** Node 2 — Authentication + Identity  
**Chat:** Chat12  
**Checkpoint:** Q2 decision lock  
**Status:** Q2 LOCKED / IMPLEMENTATION PAUSED / NEXT Q3

## 1. Purpose

This checkpoint records the completed Q2 Email-confirmation policy decision so the next session can continue directly with Q3 without reopening resolved questions.

## 2. Locked Decisions

```text
Q1 Signup / onboarding consistency = LOCKED
Q2 Email-confirmation policy      = LOCKED
Q4 One-user → one-identity        = LOCKED
```

Q1 and Q4 remain untouched by the Q2 decision.

## 3. Q2 Final Policy

Email confirmation is required before the normal authenticated user onboarding/evidence-submission flow and before Active/Usable access.

```text
Signup
 ↓
Auth User + Generic Freight Identity created atomically
 ↓
Email unconfirmed / Identity PENDING
 ↓
Email confirmed
 ↓
Authenticated evidence submission
 ↓
Verifier review
 ↓
VERIFIED + trusted_role
 ↓
ACTIVE / USABLE
```

## 4. Verification Stage Separation

The contract now distinguishes:

```text
1. Evidence submission
   → user action; requires confirmed authenticated session

2. Verifier review
   → authorized verifier action; does not require applicant session

3. Trusted-role assignment
   → server-controlled result of approved verification
```

Email confirmation is therefore a prerequisite for the normal user evidence-submission path, not a prerequisite for the verifier's own server-authorized review action.

## 5. Active Invariant

```text
Active
  = email_confirmed = true
  AND verification_status = VERIFIED
  AND trusted_role IS NOT NULL
```

Protected business access is equivalent to the Active/Usable gate.

Email confirmation alone never grants authorization.

Verification alone never grants Active access without confirmed email.

## 6. Defensive Email-Change State

```text
UNCONFIRMED + VERIFIED + trusted_role
→ allowed defensive state
→ INACTIVE
→ NO protected business access
```

This state may occur after an already-verified user's email changes and requires reconfirmation. It is not a normal onboarding state.

## 7. Independent Review

Claude reviewed the Q2 investigation and initially returned:

```text
NEEDS REVISION
```

The required corrections were incorporated into the Q2 decision report:

`03_IMPLEMENTATION/implementation_reports/Chat12_Node2_Report_Email_Confirmation_Policy.md`

The Claude review is preserved at:

`03_IMPLEMENTATION/implementation_reports/Chat12_Node2_Report_Claude_Review_Q2_Email_Confirmation_Policy.md`

Ayush approved the refined Q2 policy.

## 8. Q2 Decision Record

Primary decision report:

`03_IMPLEMENTATION/implementation_reports/Chat12_Node2_Report_Email_Confirmation_Policy.md`

Q2 status:

```text
DECIDED / LOCKED
```

## 9. Implementation Boundary

```text
Q2 architecture/policy decision = LOCKED
Implementation authorization      = NOT GRANTED
```

The Q2 lock does not authorize code, migration, Supabase configuration, RLS, or session implementation.

The exact enforcement/session mechanics remain part of the remaining Node 2 design questions.

## 10. Next Question — Q3

**NEXT: Q3 — Session lifecycle / refresh.**

Investigate:

- session establishment;
- server-side session verification;
- refresh behavior;
- logout;
- expired/invalid session behavior;
- cookie/token handling;
- protected-route behavior during refresh failure;
- interaction with the Q2 email-confirmation state machine;
- behavior when a verified user's email becomes unconfirmed after an email change.

Follow the normal workflow:

```text
Authoritative records
        ↓
Q3 investigation
        ↓
Evidence
        ↓
Decision
        ↓
Independent review where appropriate
        ↓
Ayush approval
        ↓
Q3 lock
```

Do not implement Node 2 until the complete Node 2 contract is locked.

## 11. Continuation Rule

Do not reopen Q1, Q2, or Q4 unless genuinely new evidence creates a direct conflict with the authoritative Records.
