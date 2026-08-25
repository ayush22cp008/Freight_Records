## Verdict

**APPROVE**

## 1. What is correct — all six prior concerns addressed

| Concern raised | Status | Where it's fixed |
|---|---|---|
| Live DB Active-gate must be explicit | ✅ Fixed | §3 Decision 1, §8 (explicit "never authoritative" list — JWT claims, cached middleware/client state, old session metadata) |
| JWT `email_verified` ≠ Freight verification | ✅ Fixed | §3 Decision 2 states it as an inequality directly: `JWT email_verified ≠ Freight Active` |
| Stale-state behavior needs a test | ✅ Fixed | §16, tests 7, 8, 9, 10 directly cover verification_status flipping mid-session, email unconfirming mid-session, and JWT-alone insufficiency |
| CSRF needs explicit policy | ✅ Fixed | §3 Decision 3 / §10 — SameSite + Origin validation + normal auth/authz stack, with a clear reopen condition ("if cross-origin authenticated requests are introduced, revisit with evidence") |
| Middleware matcher scope must be a deliverable | ✅ Fixed | §3 Decision 5 / §11 — explicitly deferred to implementation but scoped with requirements (protected routes covered, public/static bypass, no unnecessary auth processing) |
| PENDING/REJECTED session handling needs a decision | ✅ Fixed | §3 Decision 4 / §12 — valid session may persist, DB gate denies access, forced revocation explicitly out of scope for normal flow |

The architecture itself remains sound and correctly reflects verified Supabase/`@supabase/ssr` behavior (Middleware-only cookie refresh, `getUser()` for server verification, refresh-token rotation). Nothing in the refinement touches Q1, Q2, or Q4, and Q3 correctly remains unlocked pending your approval.

## 2. Remaining concerns (minor — don't block approval)

1. **§5 caveat is good but slightly buried.** "Do not interpret `getUser()` as an unconditional guarantee that every refresh succeeds" is exactly right and should survive verbatim into the final Node 2 contract's normative language, not just this decision report — make sure §17 contract items get worded with equal explicitness (currently item 2 is close but doesn't carry the "not unconditional" framing forward).
2. **CSRF Origin-validation mechanism is still undefined.** Decision 3 names "Origin validation" as a pillar but doesn't say whether that means checking `Origin`/`Referer` headers, a Next.js built-in, or a custom check. Not a blocker since it's correctly deferred to implementation, but flag it now so it doesn't get silently skipped later — add it as an explicit acceptance-test requirement (test 11 currently just says "enforce," not "verify Origin header is checked against expected host").
3. **Test 8 (email-unconfirm-mid-session) doesn't specify what re-triggers `email_confirmed=false`** — is this only via Supabase-initiated email-change flow, or can `verification_status` and `email_confirmed` drift independently in your DB? Worth a one-line clarification when Node 1/Q2 contract details are next touched, not a Q3 blocker.

None of these require another revision cycle — they're forward pointers for the contract-writing and implementation stages, not defects in this decision.

## 3. Exact corrections required

None mandatory for Q3 lock. Optional tightening (can be folded into §17 contract language directly, no new report cycle needed):
- Carry the "`getUser()` refresh is not unconditional" caveat into contract item 2's wording.
- Add to test 11: "Origin header, when present, is validated against the expected application host."

## 4. Final recommended Q3 policy

**Lock Q3 as Policy B (Middleware-centered session refresh with live DB Active gate)** as written in §1–§17 of this report. It correctly separates authentication (session/middleware/refresh) from authorization (Active gate/Node 1), enforces DB-liveness as the non-negotiable authority for business access, and gives CSRF/matcher/session-persistence questions concrete, defensible MVP answers with explicit reopen conditions.

## 5. Confirm

```
Q1 = LOCKED
Q2 = LOCKED
Q4 = LOCKED
Q3 = READY FOR LOCK (this review approves it; lock action itself is yours to take)
Implementation = NOT GRANTED
```

Implementation remains paused. This was review only — no code, migrations, or config touched.
