# Chat12 — Node 2 Investigation: Q2 Email-Confirmation Policy

**Project:** Freight — AI Builders Hackathon  
**Node:** Node 2 — Authentication + Identity  
**Question:** Q2 — Email-confirmation policy  
**Status:** INVESTIGATION PROMPT / NO IMPLEMENTATION AUTHORIZATION  
**Purpose:** Resolve the email-confirmation and account-activation policy before the Node 2 contract is locked.

---

## 1. Investigation Boundary

Investigate **Q2 only**.

Do **not** reopen Q1 or Q4 unless new evidence creates a genuine contradiction with the authoritative Records.

Current locked decisions:

```text
Q1 — Signup / onboarding consistency
= PostgreSQL-trigger-based atomic creation

Q4 — One-user → one-identity
= exactly 1 Freight Identity per Auth User
= DB-level UNIQUE(auth_user_id) is required
```

These are already decided and must be treated as fixed constraints for this investigation.

Do not implement any code, migration, trigger, email configuration, or route change as part of this investigation.

The output must be an **evidence-based investigation report and recommendation**, not an implementation.

---

## 2. Authoritative Records to Read First

Read these Records before investigating:

```text
00_PROJECT_CONTROL/ROADMAP.md
00_PROJECT_CONTROL/CURRENT_STATUS.md
00_PROJECT_CONTROL/PROJECT_STATE.md
00_PROJECT_CONTROL/CHECKPOINTS/Chat11_Day4_Node2_Checkpoint.md
00_PROJECT_CONTROL/CHECKPOINTS/Chat12_Day5_Node2_Checkpoint.md
02_ARCHITECTURE/Chat11_Node2_Authentication_Identity_Contract_DRAFT.md
02_ARCHITECTURE/Chat12_Node2_Signup_Onboarding_Consistency_Decision.md
03_IMPLEMENTATION/implementation_reports/Chat12_Node2_Report_Pending_Identity_vs_Verification.md
03_IMPLEMENTATION/implementation_reports/Chat12_Node2_Report_Auth_Trigger_vs_Compensation.md
03_IMPLEMENTATION/implementation_reports/Chat12_Node2_Claude_Report_Review_Q1_Q4.md
```

Also inspect any directly relevant existing Q2/email-authentication investigation records in the repository if present.

Do not rely on an old record when a newer authoritative Record conflicts with it.

---

## 3. Current Known Context

The current Node 2 Model C direction is:

```text
Auth User
    ↓
Generic Freight Identity
    ↓
requested_role = Company / Driver
verification_status = PENDING
trusted_role = NULL
    ↓
Evidence + controlled human review
    ↓
VERIFIED
    ↓
trusted_role = Company / Driver
```

The pending-identity investigation already established that identity creation occurs independently of email confirmation and that active state is intended to depend on both email confirmation and verification. Treat those as existing evidence inputs, but verify them against authoritative sources rather than assuming they are final policy.

The Q2 question remains OPEN in the Node 2 contract.

---

## 4. Questions That Must Be Answered

Investigate and answer all of the following:

### A. Is email confirmation required?

Determine whether the final Freight product should require email confirmation before an account can become active/usable.

Evaluate:

- security implications;
- account ownership assurance;
- hackathon UX implications;
- Supabase Auth capabilities and actual behavior;
- compatibility with Model C;
- compatibility with Node 1's authorization invariant.

Do not simply answer from preference. Provide evidence and rationale.

### B. What happens when email is unconfirmed?

Define the externally observable state and allowed actions for:

```text
Auth User exists
Freight Identity exists
Email unconfirmed
Verification PENDING
```

Determine:

- Can the user authenticate?
- Can the user maintain a session?
- Can the user access protected application operations?
- Can the user submit verification evidence?
- Can the verifier review the evidence?
- Can the user receive a trusted role?
- Can the user become active/usable?

Separate authentication/session existence from authorization and active-account status.

### C. Can verification proceed before email confirmation?

Determine whether:

```text
UNCONFIRMED + PENDING
        ↓
verification review
        ↓
VERIFIED
```

is allowed.

If allowed, explain why this does not create an authorization bypass.

If disallowed, explain why the product requires confirmation before verification review.

Distinguish carefully between:

- accepting/storing evidence;
- reviewing evidence;
- changing verification status;
- granting trusted role;
- granting active/usable access.

### D. When does a verified identity become active/usable?

Determine the exact activation gate.

Explicitly evaluate whether activation requires:

```text
Email confirmed
AND
Verification status = VERIFIED
AND
trusted_role != NULL
```

Also determine behavior for REJECTED identities.

### E. What is the authoritative state ordering?

Produce a final state model covering at minimum:

```text
Identity exists
Email unconfirmed
Email confirmed
Verification PENDING
Verification VERIFIED
Verification REJECTED
Trusted role NULL
Trusted role assigned
Active/usable
```

Do not collapse these states merely for implementation convenience.

---

## 5. Required State-Matrix Analysis

Produce a matrix covering at minimum:

| Email | Verification | Trusted Role | Active? | Protected Business Access? | Notes |
|---|---|---|---|---|---|
| Unconfirmed | PENDING | NULL | ? | ? | ? |
| Confirmed | PENDING | NULL | ? | ? | ? |
| Unconfirmed | VERIFIED | Driver/Company | ? | ? | ? |
| Confirmed | VERIFIED | Driver/Company | ? | ? | ? |
| Unconfirmed | REJECTED | NULL | ? | ? | ? |
| Confirmed | REJECTED | NULL | ? | ? | ? |

The investigation must fill every `?` with evidence-backed conclusions.

---

## 6. Supabase-Specific Investigation

Verify current Supabase Auth behavior from authoritative Supabase documentation and, where possible, repository configuration/source evidence.

Investigate at minimum:

- behavior of `signUp()` when email confirmation is required;
- whether an Auth User is created before confirmation;
- whether a session is returned before confirmation;
- behavior of subsequent sign-in before confirmation;
- relevant Auth user fields indicating confirmation state;
- password-reset/recovery interaction if relevant to the account-state model;
- email confirmation configuration;
- server-side methods available for checking confirmation state;
- implications of the `auth.users` INSERT trigger selected by Q1.

Do not assume hosted Supabase defaults are identical to every local/project configuration. Record configuration-dependent behavior separately from invariant platform behavior.

Use current authoritative Supabase documentation for platform claims.

---

## 7. Security Analysis

Explicitly investigate whether any proposed Q2 policy could create:

- unverified users obtaining trusted roles;
- unconfirmed users accessing protected business operations;
- role escalation through client-controlled metadata;
- verification bypass;
- account enumeration;
- session confusion;
- identity/authorization state confusion;
- inconsistent activation state;
- privilege retention after email state changes;
- rejected users regaining access.

Remember:

```text
requested_role = user intent
trusted_role   = server-controlled authorization state
```

Email confirmation must not be treated as proof of Driver/Company authorization.

Verification must not be treated as proof of email ownership.

---

## 8. Product / UX Analysis

Evaluate the practical hackathon product impact of the candidate policies.

Consider:

- signup friction;
- users who do not confirm email;
- users who confirm later;
- pending verification queues;
- whether evidence can be submitted before confirmation;
- whether reviewers can process pending evidence;
- what the user should see in each state;
- whether requiring both gates is understandable and testable.

Do not optimize UX by weakening authorization/security requirements.

---

## 9. Candidate Policies to Compare

At minimum compare these three policies:

### Policy A — Email confirmation before verification

```text
Signup
 ↓
Email confirmation
 ↓
Verification
 ↓
Active
```

### Policy B — Verification may proceed before email confirmation, but activation requires both

```text
Signup
 ↓
Identity PENDING
 ↓
Evidence / verification review may proceed
 ↓
Email confirmation + verification approval
 ↓
Active
```

### Policy C — Email confirmation not required for active application access

Evaluate this even if it appears weaker, so the report explicitly establishes why it is accepted or rejected.

You may identify another policy if evidence shows these candidates are incomplete, but do not invent a new model merely to avoid making a decision.

---

## 10. Decision Criteria

Rank candidate policies against:

1. Security
2. Authorization correctness
3. Account-ownership assurance
4. Compatibility with Model C
5. Compatibility with Q1 atomic identity creation
6. Compatibility with Q4 one-user/one-identity invariant
7. User experience
8. Implementation complexity
9. Testability
10. Hackathon suitability

Security and authorization correctness take precedence over convenience.

---

## 11. Required Output

Return a structured investigation report containing:

### 1. Executive conclusion

State the recommended Q2 policy in one clear paragraph.

### 2. Evidence collected

Separate:

```text
Verified repository evidence
Verified Supabase/platform evidence
Inference / interpretation
Unknowns
```

### 3. State model

Show the recommended state ordering as a deterministic diagram.

### 4. State matrix

Complete the matrix from Section 5.

### 5. Candidate comparison

Compare Policy A, B, and C (or any evidence-justified alternatives).

### 6. Security analysis

Explain why the recommended policy does not allow verification or authorization bypass.

### 7. Supabase behavior

Record the exact relevant Auth behavior and configuration dependencies.

### 8. Recommended Q2 decision

State:

```text
Q2 = <RECOMMENDED POLICY>
```

Then specify exact rules for:

- unconfirmed users;
- confirmation;
- verification timing;
- trusted-role assignment;
- active/usable status;
- rejected users;
- protected business access.

### 9. Contract changes required

List exactly what must change in `Chat11_Node2_Authentication_Identity_Contract_DRAFT.md` if the recommendation is approved.

Do not edit the contract as part of this investigation.

### 10. Acceptance tests required

Provide test cases that can later verify the Q2 decision.

### 11. Remaining unknowns / blockers

Only list genuine unresolved issues. Do not manufacture blockers.

### 12. Implementation status

Explicitly state:

```text
Implementation authorization = NOT GRANTED
```

---

## 12. Hard Constraints

```text
DO NOT IMPLEMENT.
DO NOT MODIFY APPLICATION CODE.
DO NOT MODIFY DATABASE MIGRATIONS.
DO NOT MODIFY SUPABASE AUTH CONFIGURATION.
DO NOT ISSUE AN IMPLEMENTATION PROMPT.
DO NOT LOCK THE NODE 2 CONTRACT.
DO NOT REOPEN Q1.
DO NOT REOPEN Q4.
DO NOT TREAT THIS REPORT AS IMPLEMENTATION AUTHORIZATION.
```

The purpose of this prompt is investigation and decision preparation only.

---

## 13. Completion Gate

The investigation is complete only when:

```text
[ ] Authoritative project-control records read
[ ] Current Node 2 contract read
[ ] Existing Q1/Q4 decisions preserved
[ ] Repository evidence inspected
[ ] Current Supabase behavior verified
[ ] Candidate policies compared
[ ] State ordering defined
[ ] State matrix completed
[ ] Security implications analyzed
[ ] Recommended Q2 policy stated
[ ] Acceptance tests proposed
[ ] Contract changes identified
[ ] Genuine unknowns recorded
[ ] Implementation remains paused
```

After this report is produced, the next workflow step is:

```text
Investigation report
        ↓
Independent review (Claude)
        ↓
Ayush approval / rejection
        ↓
Record Q2 decision
        ↓
Update Node 2 contract
        ↓
Continue to Q3
```

Do not skip the independent-review or Ayush-approval gates.