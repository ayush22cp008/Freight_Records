# Chat12 Node 2 — Q7 Investigation: Final Acceptance-Test Matrix

## 1. Executive Conclusion
The Node 2 authentication and identity model is **READY FOR REVIEW**. No irreconcilable contradictions were found between the locked decisions (Q1-Q6). The following test matrix defines the hard acceptance criteria required to prove that the system enforces the locked invariants before Node 2 implementation can be considered complete.

## 2. Final Acceptance-Test Matrix

### Q1: Signup Atomicity & 1:1 Invariant
| Test ID | Scenario | Action | Expected Result | Boundary Verified | Source |
|---|---|---|---|---|---|
| **AT-Q1-01** | Standard signup | Call `signUp` | 1 Auth User created, 1 `freight_identities` created with PENDING status | Database Atomicity | Q1 / Q4 |
| **AT-Q1-02** | Trigger failure | Simulate trigger error during `signUp` | Entire transaction rolls back; no Auth User created | Consistency | Q1 |
| **AT-Q1-03** | Duplicate identity | Attempt manual insert into `freight_identities` for existing Auth ID | Database Unique Constraint violation | 1:1 Identity Invariant | Q4 |

### Q2/Q4: Verification, Email Confirmation & Active Gate
| Test ID | Scenario | Action | Expected Result | Boundary Verified | Source |
|---|---|---|---|---|---|
| **AT-Q2-01** | Unconfirmed email | Attempt login before email confirmation | 400 Invalid Login Credentials / Email not confirmed | Platform Access | Q2 |
| **AT-Q2-02** | Confirmed but PENDING | Attempt access to Driver restricted business operation (e.g. Trips) | 403 Forbidden / Empty Data (RLS) | Active Gate / RLS | Q4 |
| **AT-Q2-03** | REJECTED status | Call restricted API route | 403 Forbidden | Authorization | Q4 |
| **AT-Q2-04** | Email change | VERIFIED user changes email, becomes unconfirmed; tries API access | 403 Forbidden (JWT `email_verified` check fails) | Active Gate freshness | Q2 |

### Q3: Session Lifecycle & Refresh
| Test ID | Scenario | Action | Expected Result | Boundary Verified | Source |
|---|---|---|---|---|---|
| **AT-Q3-01** | Expired access token | Call API with expired access token but valid refresh cookie | Middleware automatically refreshes, API returns 200 OK | Session Refresh | Q3 |
| **AT-Q3-02** | Invalid refresh token | Call API with corrupted/revoked refresh token | Middleware clears cookies, redirects to `/login` | Session Integrity | Q3 |
| **AT-Q3-03** | Logout enforcement | Call `signOut()`, then access protected route | Redirected to `/login`; server rejects cached tokens | Revocation | Q3 |

### Q5: Authentication Rate Limiting
| Test ID | Scenario | Action | Expected Result | Boundary Verified | Source |
|---|---|---|---|---|---|
| **AT-Q5-01** | Brute force login | Submit 50 invalid logins from one IP rapidly | HTTP 429 Too Many Requests | Rate Limiting (Supabase WAF) | Q5 |
| **AT-Q5-02** | Email enumeration | Request password reset for non-existent email | 200 OK generic success message | Privacy / Anti-enum | Q5 |

### Q6: RLS & Service Role Boundary
| Test ID | Scenario | Action | Expected Result | Boundary Verified | Source |
|---|---|---|---|---|---|
| **AT-Q6-01** | IDOR attempt (API) | User A sends API request with `driver_id` = User B | API uses user-scoped client; RLS blocks access or returns 403 | RLS Integrity | Q6 |
| **AT-Q6-02** | Privilege escalation | User A attempts to `UPDATE` their `trusted_role` | Database returns false/unauthorized | RLS strict `false` policy | Q6 |
| **AT-Q6-03** | Admin verification | Webhook using `service_role` updates User A to VERIFIED | Success | Service Role Quarantine | Q6 |
| **AT-Q6-04** | Table configuration | Check system catalog for RLS on core business tables | `relrowsecurity = true` for all core tables | RLS baseline | Q6 |

### Node 1 Handoff
| Test ID | Scenario | Action | Expected Result | Boundary Verified | Source |
|---|---|---|---|---|---|
| **AT-N1-01** | VERIFIED identity | VERIFIED driver calls event-logging API | API succeeds and defers to Node 1 state machine rules | Authorization Handoff | Q4 / Node 1 |
| **AT-N1-02** | Wrong role | VERIFIED Company attempts to log Driver event | API returns 403 (Trusted Role mismatch) | Role Enforcement | Q4 |

## 3. Gaps or Contradictions
- **No contradictions found.** The locked decisions (Q1 Trigger, Q2 Email First, Q3 Middleware, Q5 Supabase Rate Limiting, Q6 Strict RLS/DMZ) complement each other perfectly to form a robust, secure authentication architecture.
- **Implementation Note:** The Next.js API routes must reliably forward client IPs (e.g., via `x-forwarded-for`) so Supabase rate limiting (Q5) triggers correctly on malicious users instead of rate-limiting the Next.js server itself.

## 4. Minimum Acceptance Gate for Node 2
Before Node 2 can be marked COMPLETE:
1. The database trigger must be written and deployed.
2. The `freight_identities` table must be established.
3. Next.js Middleware must be implemented for Q3 refresh.
4. Supabase `email_confirmed` configuration must be active.
5. All AT-Q1 through AT-N1 tests must pass.

## 5. Recommendation
**READY FOR REVIEW.** 
The test matrix provides the final required guardrails for implementation. Q7 can be locked pending Ayush approval.
