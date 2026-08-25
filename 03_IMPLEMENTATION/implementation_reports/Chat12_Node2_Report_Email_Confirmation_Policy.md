# Chat12 — Node 2 Investigation: Email Confirmation Policy

## 1. Executive Conclusion
The recommended Q2 policy is **Policy A (Email confirmation before verification)**. In standard Supabase Auth, unconfirmed users cannot obtain a session JWT. Without a session, they cannot authenticate to the application to upload their verification documents. Therefore, email confirmation acts as a hard platform-level gate before the verification workflow can even begin.

## 2. Evidence Collected
- **Verified Supabase behavior:** When email confirmation is enabled, `supabase.auth.signUp()` creates the user in `auth.users` but does NOT return a session. Subsequent `signInWithPassword()` calls fail until the email is confirmed.
- **Verified repository evidence:** The `auth.users` trigger (Model C) creates the `freight_identities` record immediately upon `signUp`, so the identity exists in a `PENDING` state while the email is unconfirmed.
- **Inference:** Because unconfirmed users cannot log in, they cannot interact with the application UI to submit verification documents. Thus, the verification workflow cannot proceed until the email is confirmed.

## 3. State Model
```text
Signup
 ↓ (auth.users and freight_identities created)
Identity exists (PENDING, Unconfirmed)
 ↓ (User clicks email link)
Email confirmed (PENDING, Confirmed)
 ↓ (User logs in, uploads documents)
Evidence Submitted
 ↓ (Admin reviews)
Verification VERIFIED (Trusted role assigned)
 ↓
Active/Usable
```

## 4. State Matrix

| Email | Verification | Trusted Role | Active? | Protected Business Access? | Notes |
|---|---|---|---|---|---|
| Unconfirmed | PENDING | NULL | NO | NO | Cannot log in to submit evidence. |
| Confirmed | PENDING | NULL | NO | NO | Logged in, can submit evidence. |
| Unconfirmed | VERIFIED | Driver/Company | NO | NO | Edge case (email changed). RLS must block access if unconfirmed. |
| Confirmed | VERIFIED | Driver/Company | YES | YES | Fully active and trusted. |
| Unconfirmed | REJECTED | NULL | NO | NO | Cannot log in. |
| Confirmed | REJECTED | NULL | NO | NO | Blocked from access. |

## 5. Candidate Comparison
- **Policy A (Email confirmation before verification):** Safest and most aligned with Supabase defaults. Users must prove email ownership to log in and begin onboarding.
- **Policy B (Verification before email confirmation):** Technically infeasible without custom auth flows. Unconfirmed users have no JWT to upload evidence securely.
- **Policy C (Email confirmation not required):** Disqualified due to severe security risks (account enumeration, spam accounts, lack of ownership proof).

## 6. Security Analysis
Policy A ensures that we do not process verification documents from anonymous, unverified email addresses. It prevents spam signups from clogging the manual verification queue. RLS policies must strictly enforce `auth.jwt() -> 'email_verified' = true` (or equivalent) AND `verification_status = 'VERIFIED'` to prevent bypasses if a verified user later changes their email and becomes unconfirmed.

## 7. Supabase Behavior
- `signUp` creates the user in `auth.users` (firing the DB trigger).
- If confirmation is required, `session` is `null`.
- The JWT contains email confirmation status.
- Changing an email requires re-confirmation, meaning a VERIFIED user could temporarily lose access if their email becomes unconfirmed.

## 8. Recommended Q2 Decision
`Q2 = Policy A (Email confirmation before verification)`

- **Unconfirmed users:** Identity is `PENDING`, but they cannot log in.
- **Verification timing:** Can only occur after email confirmation and login.
- **Trusted-role assignment:** Only granted when `VERIFIED` by admin/OCR.
- **Active status:** Requires `email_confirmed = true` AND `verification_status = 'VERIFIED'`.
- **Rejected users:** Stay blocked permanently (or until re-evaluation).

## 9. Contract Changes Required
In `Chat11_Node2_Authentication_Identity_Contract_DRAFT.md`:
- Explicitly state that email confirmation is a hard prerequisite for verification document submission.
- Define "Active Account" as requiring both confirmed email and `VERIFIED` identity status.
- Specify that RLS policies must check both conditions.

## 10. Acceptance Tests Required
- **Test 1:** Attempt to submit verification documents without a valid JWT -> Fails (401).
- **Test 2:** Sign up, do not confirm email, attempt to log in -> Fails.
- **Test 3:** Confirm email, log in, attempt to access Driver protected route while PENDING -> Fails.
- **Test 4:** Admin sets `VERIFIED`, user accesses Driver route -> Succeeds.

## 11. Remaining Unknowns / Blockers
- None.

## 12. Implementation Status
`Implementation authorization = NOT GRANTED`
