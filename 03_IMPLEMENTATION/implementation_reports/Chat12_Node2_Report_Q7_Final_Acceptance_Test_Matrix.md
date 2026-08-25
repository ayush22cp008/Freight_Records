# Chat12 Node 2 — Q7 Investigation: Final Acceptance-Test Matrix

## 1. Executive Conclusion

The Node 2 authentication and identity model is **READY FOR INDEPENDENT REVIEW**. No irreconcilable contradictions were found between the locked decisions (Q1–Q6). This matrix defines the acceptance criteria required to prove that the implementation enforces the locked invariants before Node 2 can be considered complete.

Q1–Q6 remain locked and are not reopened by Q7.

## 2. Final Acceptance-Test Matrix

### Q1: Signup Atomicity & 1:1 Invariant

| Test ID | Scenario | Action | Expected Result | Boundary Verified | Source |
|---|---|---|---|---|---|
| **AT-Q1-01** | Standard signup | Execute the approved signup flow | Exactly 1 Auth User and exactly 1 `freight_identities` record are created with the expected initial status | Auth → application identity consistency | Q1 / Q4 |
| **AT-Q1-02** | Identity-creation failure | Simulate/fault the approved identity-creation mechanism | The approved atomicity/rollback guarantee is preserved; no orphan Auth User is left by the locked Q1 mechanism | Signup atomicity | Q1 |
| **AT-Q1-03** | Duplicate identity | Attempt a second `freight_identities` row for the same Auth User | DB `UNIQUE(auth_user_id)` constraint rejects the duplicate | 1:1 identity invariant | Q4 |

### Q2/Q4: Verification, Email Confirmation & Active Gate

| Test ID | Scenario | Action | Expected Result | Boundary Verified | Source |
|---|---|---|---|---|---|
| **AT-Q2-01** | Unconfirmed email | Attempt the normal protected onboarding/business flow before email confirmation | Normal protected flow is denied according to the locked Q2 email-confirmation policy | Email confirmation gate | Q2 |
| **AT-Q2-02** | Confirmed but PENDING | Authenticate successfully, then attempt a protected business operation | Session may remain valid, but live Freight Identity Active gate denies protected access with the expected 403/denial behavior | Active gate | Q2 / Q3 |
| **AT-Q2-03** | REJECTED status | Call a protected API route with a valid session | Live Freight Identity state returns REJECTED; protected business access is denied | Verification state | Q2 / Q3 |
| **AT-Q2-04** | Email state changes after verification | VERIFIED user becomes unconfirmed after an email-change/reconfirmation event, then attempts protected access | Live Freight Identity `email_confirmed = false` causes Active gate to deny protected access; JWT `email_verified` alone is never treated as the authoritative Active decision | Active-state freshness | Q2 / Q3 |
| **AT-Q2-05** | Full Active invariant | Test combinations of email confirmation, verification status, and trusted role | Protected business access is allowed only when `email_confirmed = true AND verification_status = VERIFIED AND trusted_role IS NOT NULL` | Q2 Active invariant | Q2 |

### Q3: Session Lifecycle & Refresh

| Test ID | Scenario | Action | Expected Result | Boundary Verified | Source |
|---|---|---|---|---|---|
| **AT-Q3-01** | Expired access token | Call a protected route with an expired access token and valid refresh state | Scoped Middleware performs the supported refresh flow; if refresh succeeds, the protected request continues to authentication/Active/Node 1 checks | Session refresh | Q3 |
| **AT-Q3-02** | Invalid refresh/session state | Call a protected route with invalid/revoked session state | Protected browser flow performs login handling/redirect; protected API returns 401; invalid session cookies are cleaned up where supported | Session integrity | Q3 |
| **AT-Q3-03** | Logout enforcement | Execute supported sign-out, then access a protected route | Usable session is cleared and subsequent protected access fails authentication | Logout | Q3 |
| **AT-Q3-04** | Stale token after state change | Keep a valid session/token while Freight Identity changes from Active to REJECTED or unconfirmed | Next protected request checks live Freight Identity state and denies access; stale JWT/session metadata cannot preserve Active access | Live Active gate | Q2 / Q3 |
| **AT-Q3-05** | Middleware scope | Exercise public, static, protected page, and protected API routes | Protected routes are covered; public/static routes do not incur unnecessary protected-session processing | Middleware boundary | Q3 |

### Q5: Authentication Rate Limiting

| Test ID | Scenario | Action | Expected Result | Boundary Verified | Source |
|---|---|---|---|---|---|
| **AT-Q5-01** | Sustained authentication abuse | Send authentication traffic beyond the currently documented/configured Supabase Auth limit for the tested endpoint | Underlying Auth operation returns `429 Too Many Requests` according to the current project/platform configuration | Supabase Auth native rate limiting | Q5 |
| **AT-Q5-02** | Legitimate authentication | Perform normal authentication while under the configured quota | Legitimate request is processed normally | Availability under quota | Q5 |
| **AT-Q5-03** | Email enumeration | Request password recovery for an account/non-account scenario | Responses remain generic/non-enumerating according to the approved Auth flow | Privacy / anti-enumeration | Q5 |
| **AT-Q5-04** | Client-IP forwarding | Exercise the deployed Next.js/proxy path used for Auth requests | If `Sb-Forwarded-For` is required, it is derived from a trusted proxy/client-IP source; an untrusted client cannot directly choose the value | Secure rate-limit identity | Q5 |
| **AT-Q5-05** | 429 handling | Trigger a real configured rate limit | Application preserves the underlying rate-limit meaning and does not convert 429 into a misleading authentication/authorization result | Failure handling | Q5 |

**Q5 rule:** tests must use the actual current Supabase project configuration and documented endpoint behavior. No guessed request count is an architectural constant.

### Q6: RLS & Service-Role Boundary

| Test ID | Scenario | Action | Expected Result | Boundary Verified | Source |
|---|---|---|---|---|---|
| **AT-Q6-01** | IDOR attempt | User A requests User B's protected resource through normal user access | RLS denies/filters the cross-user row; Node 1 authorization cannot bypass RLS | RLS + IDOR | Q6 / Node 1 |
| **AT-Q6-02** | Privilege escalation | User A attempts to update `trusted_role` or `verification_status` through ordinary user access | Database/RLS denies the unauthorized mutation | Security-sensitive state protection | Q6 |
| **AT-Q6-03** | Authorized privileged verification | Explicitly authorized trusted server workflow performs an approved verification mutation using `service_role` | Operation succeeds only after server-side authentication/authorization and produces the required audit record | Privileged boundary | Q6 |
| **AT-Q6-04** | RLS baseline | Inspect protected business tables | RLS is enabled and expected CRUD policies exist | RLS baseline | Q6 |
| **AT-Q6-05** | FORCE RLS / owner bypass | Inspect protected table owner role and `FORCE ROW LEVEL SECURITY` status, then verify the approved architecture exception if FORCE RLS is not used | Every protected table either has FORCE RLS enabled or a documented/proven architecture preventing untrusted access through the table-owner role | Table-owner boundary | Q6 |
| **AT-Q6-06** | Privileged audit | Perform a privileged mutation of `verification_status`, `trusted_role`, or administrative approval state | Mandatory audit record contains actor identity, target resource, and timestamp and cannot be modified by ordinary users | Auditability | Q6 |
| **AT-Q6-07** | Service-role import boundary | Run the repository lint/CI service-role allowlist check against an approved and an unapproved import | Approved server path passes; unapproved user-facing import fails the automated check | Privileged code boundary | Q6 |
| **AT-Q6-08** | SECURITY DEFINER review | Inspect any Auth trigger/function using `SECURITY DEFINER` | Execution role, `search_path`, privileges, callable surface, and function-body writes are explicitly verified as safe | Trigger/function privilege boundary | Q6 |
| **AT-Q6-09** | Service-role compromise procedure | Simulate/confirm credential exposure | Documented response immediately rotates/restricts the credential, audits the exposure window, identifies affected resources, and records remediation | Secret compromise response | Q6 |

### Node 1 Authorization Handoff

| Test ID | Scenario | Action | Expected Result | Boundary Verified | Source |
|---|---|---|---|---|---|
| **AT-N1-01** | RLS passes, Node 1 denies | Use a row the user may access under RLS but request a business operation forbidden by Node 1 state-machine rules | Operation returns 403/denial | RLS vs Node 1 separation | Node 1 / Q6 |
| **AT-N1-02** | RLS blocks, Node 1 would allow | Attempt a cross-user resource/action that business rules might otherwise permit | RLS blocks the data access before the operation can proceed | Database boundary | Q6 / Node 1 |
| **AT-N1-03** | VERIFIED Driver | VERIFIED Driver calls an event-logging operation permitted by Node 1 | API succeeds only after authentication, Active gate, and Node 1 authorization | Authorization handoff | Node 1 / Q2 / Q3 |
| **AT-N1-04** | Wrong role | VERIFIED Company attempts a Driver-only operation | API returns 403 | Role enforcement | Node 1 |

## 3. Gaps / Contradictions

No architectural contradictions were found between Q1–Q6.

The following are **implementation verification requirements**, not unresolved architecture decisions:

1. Exact current route/middleware matcher must be derived from the real application route tree.
2. Exact current Supabase Auth rate-limit configuration and endpoint behavior must be verified at implementation time; Q5 deliberately avoids hard-coded guessed values.
3. The deployed proxy path must establish a trusted client-IP source before any `Sb-Forwarded-For` forwarding is used.
4. Exact protected-table RLS policy coverage, owner roles, and FORCE RLS status must be verified.
5. The approved service-role import allowlist and automated lint/CI mechanism must be implemented and tested.
6. Any `SECURITY DEFINER` Auth trigger/function must undergo the locked Q6 security review.

These checks do not reopen Q1–Q6 unless new evidence creates a genuine architectural conflict.

## 4. Minimum Acceptance Gate for Node 2

Before Node 2 can be marked **COMPLETE**:

1. Q1–Q6 locked invariants are implemented without contradiction.
2. The approved signup/identity-creation mechanism proves the Q1 atomicity requirement.
3. `freight_identities` exists with the Q4 one-to-one uniqueness constraint.
4. Q2 Active enforcement is based on current email-confirmation + verification + trusted-role state.
5. Q3 Middleware/session lifecycle is implemented and protected routes are correctly covered.
6. Q5 Supabase Auth native rate limiting is verified against the actual current configuration, including correct 429 behavior and secure client-IP handling where required.
7. Q6 RLS/service-role controls are verified, including FORCE RLS/table-owner handling, privileged audit logging, SECURITY DEFINER review, and service-role import allowlist enforcement.
8. All Q1–Q6 acceptance tests in this matrix pass, including the RLS-only and Node 1-only separation tests.
9. All Node 1 authorization handoff tests pass.
10. Evidence for each acceptance test is recorded before declaring Node 2 implemented/complete.

## 5. Recommendation

**READY FOR INDEPENDENT REVIEW.**

Q7 now accurately tests the locked Q1–Q6 decisions without reopening them or introducing guessed platform behavior.

Next workflow:

```text
Corrected Q7 matrix
        ↓
Independent review
        ↓
Ayush approval
        ↓
Q7 LOCK
        ↓
Node 2 decision work COMPLETE
        ↓
Authentication implementation / acceptance according to the locked Node 2 contract
```

**No implementation is authorized by this Q7 report.**
