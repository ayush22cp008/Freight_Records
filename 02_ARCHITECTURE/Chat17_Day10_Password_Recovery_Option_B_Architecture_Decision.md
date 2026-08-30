# Chat17 — Day 10 — Password Recovery — Option B Architecture Decision

## Status
APPROVED / LOCKED ARCHITECTURE DIRECTION

## Decision
Ayush selected and approved **Option B — token-hash-based recovery** as the password-recovery architecture direction for the Freight application.

## Why Option B

The current production flow uses Supabase Auth and `@supabase/ssr`, while the existing recovery callback mixes PKCE behavior with a `token_hash`/`verifyOtp()` implementation. A fresh recovery email clicked once has been manually verified to fail with `otp_expired` / `Invalid or expired recovery link`.

Supabase documents `{{ .TokenHash }}` as a supported email-template variable for constructing a custom recovery link, and documents email-link prefetching as a cause of single-use confirmation/recovery links being consumed before the user clicks them. A custom application link can place an intermediate application page in front of the single-use verification action. citeturn0search0turn0search10

## Approved Recovery Flow

```text
Forgot Password
    ↓
resetPasswordForEmail()
    ↓
Supabase recovery email
    ↓
Custom Freight recovery link
    ↓
Freight recovery page / explicit user action
    ↓
Verify recovery token_hash
    ↓
Valid recovery session
    ↓
Set New Password
    ↓
Confirm New Password
    ↓
updateUser()
    ↓
Password changed successfully
    ↓
Login
```

## Domain Decision

No custom/purchased domain is required for the application recovery URL.

The production application domain already available to the project is:

```text
https://freighthackathon.vercel.app
```

This Vercel deployment domain can be used for the recovery application URL and must be allowlisted in Supabase Authentication URL Configuration for the exact recovery redirect used by the implementation. Supabase documents that redirect URLs passed to authentication methods must be in the project's allowed redirect list. citeturn0search6

A separate custom email-sending domain is **not a prerequisite for the application URL itself**.

## Important Implementation Dependency — SMTP / Template Editing

The current Supabase dashboard screenshot shows that the hosted Free-tier project's default email provider does not currently expose template editing and instead offers **Set up custom SMTP**.

Current Supabase documentation states that new Free-tier projects using Supabase's default email provider cannot modify hosted Auth email templates; template customization is available when a custom SMTP provider is configured (or on paid plans). citeturn0search5turn0search2

Therefore Option B is architecturally locked, but **implementation is blocked until the project has a supported way to customize the Reset Password email template**.

This does NOT mean Ayush must purchase a custom domain. It means we must choose/configure a supported SMTP/template-editing path before changing the recovery email template.

## Do Not Do Yet

- Do not change the Supabase Reset Password template yet.
- Do not enable the Password Changed notification merely to fix recovery.
- Do not remove `/api/auth/confirm` until the replacement flow is implemented and verified.
- Do not add a new permanent reset-token system.
- Do not add a separate Change Password feature.
- Do not begin Reviewer Authentication implementation as part of this decision.

## Locked Password Recovery UX

The previously approved decisions remain in force:

```text
Email reset link
→ Directly reach password recovery flow
→ New Password + Confirm Password
→ Use project/Supabase password policy
→ Invalid/expired recovery state gives safe recovery path
→ Successful reset shows success and returns to Login
→ Password only is changed
→ Same mechanism supports Company, Driver, Reviewer
→ Neutral account-existence response
→ Existing provider/project rate limits preserved
→ No separate Change Password feature in this Subnode
```

## Security Boundary

Password recovery must not change:

```text
Application role
Application identity
Verification status
Reviewer authorization
Business data
```

The recovery credential must be verified before `updateUser({ password })` is allowed.

## Evidence / Related Records

Fresh manual failure investigation:
`05_DEBUGGING/investigations/Chat17_Day10_Subnode_Password_Recovery_Fresh_Link_Failure_Investigation_Report.md`

Approved general password-recovery decisions:
`02_ARCHITECTURE/Chat17_Day10_Password_Recovery_Decisions.md`

Original implementation plan:
`03_IMPLEMENTATION/plans/Chat17_Day10_Subnode_Reviewer_Login_Password_Recovery_Implementation_Plan.md`

Original implementation prompt:
`03_IMPLEMENTATION/prompts/Chat17_Day10_Subnode_Reviewer_Login_Password_Recovery_Implementation_Prompt.md`

## Next Required Stage

Before implementation, resolve the **email-template customization / SMTP dependency** and then create a targeted implementation prompt through the GitHub Records bridge.

No source-code implementation is authorized by this architecture record alone.
