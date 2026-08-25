Let me verify the Supabase rate-limit claims against current docs before giving a verdict.Now I have the current doc text against which to check Q5's claims. Here's the independent review.

## Verdict

**Q5 is directionally correct and can be locked — but two factual items in the report need correction before locking, and the report claims a public example number that current docs no longer state that way.**

Supabase-native rate limiting as MVP primary control: **correct**. Rejecting hard account lockout: **correct**. Rejecting a custom Redis/Upstash limiter for MVP: **correct** and well-reasoned. The Q2/Q3/Node1 boundary preservation: **correct, cleanly done**. The acceptance-test list is strong. The problems are narrower — specific numeric/mechanism claims.

## Problems / concerns (checked against current Supabase docs, fetched today)

**1. Sign-in is NOT IP-rate-limited by Supabase — Q5's §6 table is wrong on this point.**
The current docs table has no separate "sign-in" row limited by IP. What Supabase actually rate-limits:
- `/auth/v1/signup` (and `/recover`, `/user` email-change) — limited **project-wide**, 2 emails/hour on built-in SMTP, or per-user 60s cooldown for the signup-confirmation-send action itself.
- `/auth/v1/token` (this **is** how login/password sign-in and refresh flow through) — IP-limited, but the doc lists it as "Token refresh requests," 1800/hour.
- There is no documented endpoint literally called "sign-in: IP-based quota of 30 requests per 5-minute interval." That number pattern (30 per bucket) matches the **general token-bucket capacity** (bucket max = 30, refills at the configured rate), not a distinct "sign-in" quota entry.

Q5's §6 line — *"sign-in/sign-up: IP-based quota of 30 requests per 5-minute interval by default"** — is **not** what the current docs state and should not be locked as-is. It conflates the bucket capacity (30) with a rate (per 5 min) that isn't in the table. This is exactly the kind of hard-coded, unverified numeric claim the report itself warns against elsewhere (§6, §16) — it's an internal inconsistency: the report simultaneously says "don't hard-code Supabase defaults" and then hard-codes an inaccurate one.

**2. Password-reset/signup-confirmation cooldown claim needs a caveat.**
Correct that it's 60s "last request of the user" cooldown — but note this is **per-user**, not IP-based, and it's *customizable*. Q5 states this correctly elsewhere (§6 bullet) but the table entry in §7 blurs "cooldown/quota" generically, which is fine for a state matrix but should not be read as IP-based.

**3. MFA challenge/verify limit (15/hour, IP-based) is absent from Q5's evidence list entirely.**
Not wrong, just incomplete — worth adding since MFA is in-scope of "authentication endpoints" per the threat model in §3. Low severity since Freight's MVP likely doesn't use MFA yet, but the contract in §17 should note this endpoint exists and is unaddressed rather than silently omitting it.

**4. `Sb-Forwarded-For` handling is correctly described** — secret API key requirement, explicit opt-in, no trusting raw client `x-forwarded-for` — this matches current docs exactly, including the correct point that publishable/anon/service_role keys are *not* supported for this header. Good catch, no correction needed here.

**5. Hard-lockout rejection is correct and standard** (OWASP-aligned: lockout DoS > brute-force risk in most consumer contexts). No issues.

## Exact corrections required before lock

In §6 (Threshold Policy → MVP), replace:

> sign-in/sign-up: IP-based quota of 30 requests per 5-minute interval by default;

with:

> sign-in and token operations flow through `/auth/v1/token`, which is IP-limited to 1800 requests/hour (token-bucket, burst capacity up to 30); there is no separate documented "sign-in" quota distinct from token-refresh handling. Signup/email-triggering endpoints (`/auth/v1/signup`, `/recover`, `/user`) are limited project-wide to 2 emails/hour (built-in SMTP) plus a 60-second per-user cooldown on repeat sends.

Add to §6 / §17 evidence list:

> MFA challenge/verify (`/auth/v1/factors/:id/challenge`, `/verify`) is IP-limited to 15 requests/hour — out of scope for current MVP (no MFA implemented) but should be re-verified if MFA is added later.

Everything else in §6, §7, §16 already correctly hedges as "must re-verify at implementation time" — keep that framing; just fix the one concrete wrong number rather than removing the hedge.

## Are acceptance tests and security policy sufficient?

Yes, with the addition of one test: verify that the actual sign-in flow (`/auth/v1/token?grant_type=password`) is the endpoint being IP-throttled at 1800/hr with burst-30, not an assumed nonexistent lower "sign-in" quota — so the 429 behavior the team expects to see in testing matches the endpoint actually hit. Add as test #16.

## Does Q5 preserve Q2/Q3/Node1 boundaries?

Yes — §13 is clean. Rate-limit pass ≠ Active, ≠ authorized session, ≠ Node 1 clearance. No boundary violations. Request ordering in §9 is correctly sequenced (rate limit → auth → session → Q2 → Node1).

## Final Q5 recommendation

**Lock Q5 as written, after correcting the §6 sign-in quota line (item 1 above) and adding the MFA endpoint note (item 3).** The underlying policy decision (Supabase-native, no custom limiter, no lockout, generic responses, verified `Sb-Forwarded-For` handling) is sound and appropriately hedges configuration-dependent numbers everywhere except that one line. Fix the one factual claim, add test #16, then proceed to lock and move to Q6.
