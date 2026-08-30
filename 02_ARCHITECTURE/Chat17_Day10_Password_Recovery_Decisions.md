# Chat17 — Day 10 — Password Recovery Decisions

## Status
APPROVED / LOCKED FOR IMPLEMENTATION

## Scope
This record captures the password-recovery decisions approved by Ayush for the Chat17 Day 10 Node 2 Subnode.

Reviewer authentication is intentionally deferred until the password-recovery work is settled and verified.

## Approved Decisions

| Decision | Final choice |
|---|---|
| Recovery method | Email reset link |
| After clicking reset link | Directly open Set New Password |
| Password confirmation | Require New Password + Confirm Password |
| Password requirements | Use Supabase/project-configured password policy |
| Invalid/expired reset link | Show error and provide Request New Reset Link / Login path |
| After successful reset | Show success, then return user to Login |
| What password reset changes | Password only |
| Supported roles | Company + Driver + Reviewer |
| Account enumeration | Use neutral recovery-request response |
| Multiple reset requests | Preserve existing provider/project rate-limit controls |
| Logged-in user password change | Only through Forgot Password in this Subnode; no separate Change Password feature |
| Recovery-link reuse | Follow Supabase recovery/session semantics; do not create reusable permanent reset tokens |

## Locked Password-Reset Invariants

A password reset must not change:

```text
Application role
Application identity
Verification status
Reviewer authorization
Company/Driver/Reviewer account state
```

Password recovery is an authentication credential recovery operation only.

## Approved End-to-End Flow

```text
Freight Login
    ↓
Forgot Password?
    ↓
Enter Email
    ↓
Neutral Response
    ↓
Supabase Recovery Email
    ↓
Reset Password link
    ↓
Secure Recovery Session
    ↓
Set New Password
    ├── New Password
    └── Confirm Password
    ↓
Update Password
    ↓
Password Changed Successfully
    ↓
Login
    ↓
Normal role-aware application routing
```

## Security Requirements

- Do not disclose account existence through recovery-request responses.
- Do not treat an email address or arbitrary URL parameter as authorization to change a password.
- Invalid or expired recovery state must not permit a password change.
- Do not expose service-role credentials or secrets to the client.
- Do not bypass application authorization or verification gates after reset.
- Do not add an administrator password override.
- Do not add Driver Code / Driver ID as an authentication credential.

## Role Coverage

The same password-recovery mechanism is intended for:

```text
Company
Driver
Reviewer
```

The mechanism must not create separate password stores or separate authentication systems for each role.

## Scope Boundary

This record does not authorize reviewer-authentication implementation by itself. Reviewer authentication/routing remains the next separate workstream after password-recovery behavior is settled and verified.

## Source / Related Records

Investigation:
`05_DEBUGGING/investigations/Chat17_Day10_Subnode_Reviewer_Login_Password_Recovery_Investigation_Report.md`

Implementation plan:
`03_IMPLEMENTATION/plans/Chat17_Day10_Subnode_Reviewer_Login_Password_Recovery_Implementation_Plan.md`

Implementation prompt:
`03_IMPLEMENTATION/prompts/Chat17_Day10_Subnode_Reviewer_Login_Password_Recovery_Implementation_Prompt.md`

## Approval

Ayush explicitly approved the recommendations for Q1–Q10 and Q12, and selected Q11 option A: password changes in this Subnode are available only through Forgot Password; no separate Change Password feature is added.
