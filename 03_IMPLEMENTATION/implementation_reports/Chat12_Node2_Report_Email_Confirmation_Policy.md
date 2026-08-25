# Chat12 — Node 2 Decision Report: Email Confirmation Policy

## 1. Final Decision

**Q2 = LOCKED — Email confirmation is required before normal authenticated onboarding and Active/Usable access.**

The policy is refined as follows:

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

Email confirmation is a prerequisite for the normal authenticated user flow and therefore for user evidence submission. It is not itself a prerequisite for an authorized verifier to review or approve evidence as a server-side/admin action.

## 2. Evidence Basis

### Verified Supabase behavior

With Supabase Confirm Email enabled, signup creates the Auth user but does not return a session. The user must confirm the email before normal sign-in. Supabase also exposes `email_confirmed_at` on the Auth user; a null value means the email is not confirmed.

### Verified project evidence

The selected Model C architecture creates the generic Freight Identity immediately with the Auth User transaction, so identity existence is independent from email confirmation.

### Independent review

Claude independently reviewed the Q2 investigation and returned **NEEDS REVISION**. The core Policy A direction was accepted, but the review identified four material precision issues:

1. “Verification before confirmation is technically infeasible” was too strong. An out-of-band server/admin workflow could technically receive or review evidence before confirmation, but that is excluded by Freight policy for the MVP.
2. “Verification” was ambiguous and must be separated into evidence submission, verifier review, and trusted-role assignment.
3. `UNCONFIRMED + VERIFIED` required an explicit policy. It is allowed only as a defensive inactive state, principally for an already-verified user whose email becomes unconfirmed after an email change.
4. An `email_verified` JWT claim must not be treated as the sole authoritative enforcement mechanism for active access because token freshness must be considered. The exact enforcement mechanism remains an implementation-boundary question for later Node 2 work.

Ayush approved these corrections and the refined Q2 policy.

## 3. Exact Q2 Policy

### Email confirmation

Email confirmation is required before the user can obtain the normal authenticated session used for onboarding and evidence submission.

### Evidence submission

A user must have a confirmed email and authenticated session before submitting verification evidence through the normal Freight UI/API flow.

### Verifier review

Verifier review is a separate server-authorized action. It does not depend on the applicant's current session or email-confirmation state.

### Trusted-role assignment

A trusted role may be assigned only after authorized verification approval. `requested_role` remains user intent and is never authorization.

### Active / usable account

A Freight account is Active/Usable only when all three conditions hold:

```text
email_confirmed = true
AND
verification_status = VERIFIED
AND
trusted_role IS NOT NULL
```

Protected business access is equivalent to the Active/Usable gate.

### Rejected users

`REJECTED` identities do not receive a trusted role and do not receive protected business access.

## 4. Verification Stage Separation

The contract must treat verification as three distinct stages:

```text
1. Evidence submission
   → user action; requires confirmed authenticated session

2. Verifier review
   → authorized verifier action; does not require applicant session

3. Trusted-role assignment
   → server-controlled result of approved verification
```

This prevents email confirmation from being incorrectly treated as a prerequisite for the verifier's own administrative action.

## 5. State Model

```text
AUTH USER + IDENTITY CREATED
        ↓
EMAIL UNCONFIRMED / PENDING
        ↓
EMAIL CONFIRMED / PENDING
        ↓
EVIDENCE SUBMITTED
        ↓
VERIFIER REVIEW
        ↓
VERIFIED + TRUSTED ROLE
        ↓
ACTIVE / USABLE
```

Rejected path:

```text
PENDING
   ↓
REJECTED
   ↓
NO TRUSTED ROLE
   ↓
NO PROTECTED BUSINESS ACCESS
```

Defensive email-change path:

```text
ACTIVE
   ↓
email changes / requires reconfirmation
   ↓
UNCONFIRMED + VERIFIED + trusted_role
   ↓
INACTIVE / NO PROTECTED BUSINESS ACCESS
   ↓
email re-confirmed
   ↓
ACTIVE again, subject to the final session/enforcement mechanism
```

`UNCONFIRMED + VERIFIED` is therefore an allowed but inactive defensive state, not a normal onboarding state.

## 6. State Matrix

| Email | Verification | Trusted Role | Active? | Protected Business Access? | Meaning |
|---|---|---|---|---|---|
| Unconfirmed | PENDING | NULL | NO | NO | New identity awaiting confirmation |
| Confirmed | PENDING | NULL | NO | NO | Authenticated onboarding / evidence stage |
| Unconfirmed | VERIFIED | Driver/Company | NO | NO | Defensive post-email-change state |
| Confirmed | VERIFIED | Driver/Company | YES | YES | Fully active trusted account |
| Unconfirmed | REJECTED | NULL | NO | NO | Rejected / unconfirmed |
| Confirmed | REJECTED | NULL | NO | NO | Rejected |

## 7. Security Invariants

```text
Active
  = email_confirmed = true
  AND verification_status = VERIFIED
  AND trusted_role IS NOT NULL
```

```text
trusted_role IS NOT NULL
  requires
verification_status = VERIFIED
```

```text
Protected Business Access
  ↔ Active
```

Email confirmation alone never grants authorization.

Verification alone never grants Active access without confirmed email.

Client-controlled `requested_role`, metadata role, `trusted_role`, or `verification_status` never constitutes authorization.

The final implementation must not rely on a potentially stale email-confirmation JWT claim alone as the authoritative Active-access guarantee. Exact enforcement and session-refresh mechanics remain implementation-design work for later Node 2 questions.

## 8. Candidate Policy Decision

### Policy A — Email confirmation before normal authenticated onboarding

**SELECTED.**

This provides the strongest ownership-assurance boundary while fitting the Supabase Auth model and the hackathon MVP.

### Policy B — Verification before email confirmation

**NOT SELECTED as Freight policy.**

An out-of-band/admin path could technically process evidence before confirmation, but the MVP does not need this path and allowing it would weaken the intended email-ownership boundary.

### Policy C — No email confirmation requirement

**REJECTED.**

It weakens account-ownership assurance and is not appropriate for the Freight authorization model.

## 9. Acceptance Tests

1. Signup with confirmation enabled creates Auth User + Freight Identity but no normal authenticated session.
2. Unconfirmed user cannot sign in through the normal password flow.
3. Confirmed user can authenticate but cannot access protected Driver/Company operations while PENDING.
4. Confirmed user can submit verification evidence.
5. Authorized verifier can review/approve a pending identity without depending on the applicant's session.
6. VERIFIED + trusted role + confirmed email permits Active/Usable access.
7. VERIFIED + trusted role + unconfirmed email does not permit Active/Usable or protected business access.
8. PENDING or REJECTED identity cannot obtain protected business access through client-controlled fields.
9. A trusted role cannot exist without `verification_status = VERIFIED`.
10. Email confirmation alone cannot grant trusted role or Active status.
11. Email change/reconfirmation behavior must preserve the invariant: an unconfirmed verified account is inactive until confirmation is restored.
12. The eventual enforcement mechanism must prevent stale-session state from granting protected access after email confirmation state changes.

## 10. Contract Changes Required

Update Section 11 of:

`02_ARCHITECTURE/Chat11_Node2_Authentication_Identity_Contract_DRAFT.md`

to incorporate the final Q2 rules in this report.

The contract must not prescribe a specific JWT/RLS implementation solely from Q2; exact enforcement belongs to the remaining Node 2 security/session design work.

## 11. Independent Review Result

Claude review:

```text
Initial review = NEEDS REVISION
Core Policy A = ACCEPTED
Required refinements = COMPLETED
```

Ayush decision:

```text
APPROVED
```

## 12. Final Status

```text
Q2 Email-confirmation policy = LOCKED
Q1 Signup consistency        = LOCKED
Q4 One-user/one-identity     = LOCKED
Implementation authorization = NOT GRANTED
```

The Q2 decision is now final at the architecture/policy level. Implementation remains paused until the remaining Node 2 questions are resolved and the complete Node 2 contract is locked.