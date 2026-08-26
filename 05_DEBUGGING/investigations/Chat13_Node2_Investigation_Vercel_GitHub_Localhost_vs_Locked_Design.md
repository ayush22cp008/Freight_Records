# Chat13 — Node 2 Investigation: Vercel vs GitHub vs Localhost vs Locked Design

## Status

**Purpose:** Evidence-driven reconciliation before Node 2 implementation

**Hackathon Day:** Day 6

**Chat:** Chat13

**Node:** Node 2 — Authentication + Identity

**Source repository under investigation:** `ayush22cp008/freight_hackathon`

**Records repository:** `ayush22cp008/Freight_Records`

**Production/Vercel URL recorded by source repo metadata:** `https://freighthackathon.vercel.app`

**Investigation only. No implementation is authorized by this file.**

---

## 1. Objective

Determine, with evidence, why the Vercel/production application and localhost application currently show different authentication UI/behavior; establish which code state each environment represents; compare both against the locked Node 2 Q1–Q7 architecture; and determine the correct target behavior before implementation.

This investigation must not assume that:

- Vercel is correct merely because it currently works;
- localhost is correct merely because it contains newer local changes;
- GitHub source equals the currently deployed Vercel artifact without commit/deployment evidence;
- source migrations are applied to the deployed database merely because they exist in code;
- visual similarity or difference proves backend correctness.

---

## 2. Safety Baseline

Until this investigation is complete:

```text
DO NOT PUSH LOCAL CHANGES
DO NOT COMMIT LOCAL CHANGES
DO NOT RUN RESET/DESTRUCTIVE GIT COMMANDS
DO NOT APPLY NEW DATABASE MIGRATIONS
DO NOT ALTER PRODUCTION DATABASE
DO NOT ALTER VERCEL SETTINGS
DO NOT MODIFY APPLICATION SOURCE CODE
```

Preserve the currently deployed state as the operational baseline while collecting evidence.

If any investigation step would mutate production/shared state, stop and report rather than proceeding.

---

## 3. Authoritative Sources

Use these sources in this order for their respective roles:

1. **Locked Node 2 architecture/decisions:** latest approved records in `Freight_Records`.
2. **Application source:** `ayush22cp008/freight_hackathon` GitHub repository.
3. **Local filesystem:** actual localhost working tree, inspected by Antigravity.
4. **Vercel/production:** actual deployment behavior and deployment metadata where safely verifiable.
5. **Database state:** only from direct evidence; do not infer from source files.

The Records repository is the coordination/source-of-truth layer for project decisions; the source repository is the code source of truth for what is supposed to ship.

---

## 4. Locked Node 2 Requirements to Compare Against

Do not reopen Q1–Q7 unless concrete evidence reveals a genuine architectural conflict.

### Q1 — Signup consistency

The final architecture requires atomic Auth User → generic Freight Identity creation, using the approved PostgreSQL/Auth-trigger direction, with no durable Auth User lacking its required Freight Identity.

### Q2 — Email confirmation / Active gate

Normal authenticated onboarding/evidence submission requires confirmed email.

Protected business access requires:

```text
email_confirmed = true
AND verification_status = VERIFIED
AND trusted_role IS NOT NULL
```

Email confirmation alone is not authorization.

### Q3 — Session lifecycle / refresh

Middleware-centered session refresh is the approved direction, using the supported Supabase SSR boundary.

Protected business access must use live Freight Identity DB state for Active evaluation.

A JWT `email_verified` claim alone is not authoritative for Freight Active access.

Session validity, Active status, and Node 1 operation authorization are separate layers.

### Q4 — One user / one identity

Exactly one Freight Identity per Auth User must be enforced at the database level.

### Q5 — Authentication rate limiting

MVP uses Supabase-native Auth rate limiting; no custom Redis/Upstash/distributed limiter unless a new evidence-based decision authorizes one.

### Q6 — RLS / service-role boundary

Normal user-facing database access should use the authenticated user-scoped path and RLS.

`service_role` is a server-only privileged capability for narrowly approved operations; it is not an authorization mechanism and must not be used as a shortcut for ordinary user CRUD.

### Q7 — Acceptance matrix

The latest approved Node 2 acceptance matrix is the final acceptance reference.

---

## 5. Environment Identity — MUST VERIFY

### A. GitHub source

Repository:

```text
ayush22cp008/freight_hackathon
```

Default branch:

```text
main
```

The repository metadata currently identifies:

```text
homepage = https://freighthackathon.vercel.app
```

Record the exact current `main` commit SHA at investigation time.

### B. Vercel / Production

Verify, where safely available:

- production URL;
- deployment commit SHA;
- branch/ref used by deployment;
- deployment timestamp;
- whether the deployment corresponds to current `origin/main`;
- whether a previous commit is being served.

Do not infer deployment commit from the URL alone.

### C. Localhost

Record:

- local repository path;
- local HEAD SHA;
- `git status` summary;
- `git diff --stat`;
- untracked files;
- whether local is ahead/behind/diverged from `origin/main`.

### D. Database

Where direct verification is possible, record whether local and production point to the same Supabase project/database.

Do not print credentials or secret values.

---

## 6. Screen / Behavior Reconciliation

Use the supplied screenshots as visual evidence that the same logical authentication page is different between environments, but do not treat screenshots alone as proof of backend correctness.

For each visible difference, record:

```text
Vercel behavior
Localhost behavior
Exact UI difference
Likely source path
Backend/API dependency
Node 2 relevance
VERIFIED / INFERRED / UNKNOWN
```

Known difference that must be explicitly checked:

```text
Vercel/GitHub:
Email-oriented login / manual Driver Code flow

Localhost:
Driver ID / auto-generated Driver Code flow
```

Determine whether this difference comes from source code, database schema, environment configuration, stale deployment, or a combination.

---

## 7. Source Comparison — GitHub vs Localhost

Reconcile all relevant authentication/identity files, not only the already observed examples.

At minimum inspect:

```text
src/app/api/auth/signup/route.ts
src/app/api/auth/login/route.ts
src/app/api/auth/logout/route.ts
src/app/login/page.tsx
src/app/signup/page.tsx
middleware / proxy-related files
Supabase browser/server helpers
identity-related modules
verification-related modules
src/db/migrations/**
package.json / relevant auth dependencies
```

For every relevant difference classify:

```text
Same
GitHub-only
Local-only
Modified
Untracked
Deleted
```

Record exact paths and concise semantic differences. Do not dump full source files into the report.

---

## 8. Authentication Flow Verification

### GitHub/Vercel flow

Verify the actual flow from source:

```text
Signup
→ Auth User creation
→ application identity/driver creation
→ session behavior

Login
→ credential input
→ Auth lookup
→ Supabase authentication
→ session creation

Protected request
→ session check
→ identity resolution
→ authorization
```

### Localhost flow

Verify the actual flow from source:

```text
Signup
→ generated Driver Code?
→ Auth User creation
→ application identity/driver creation
→ response/session

Login
→ Driver ID/Driver Code
→ lookup
→ email derivation?
→ Supabase authentication
→ session creation
```

Do not assume the local Driver ID implementation is correct simply because it works locally.

---

## 9. Q1 / Identity Consistency Investigation

Verify whether either environment currently satisfies:

```text
ONE Auth User
     ↓
ONE Freight Identity
```

Determine:

- whether `freight_identities` exists;
- whether `auth_user_id` is unique;
- whether Auth User and application identity creation is atomic;
- whether signup can leave an orphan Auth User;
- whether concurrent signup can create duplicate identities;
- whether the current `drivers` table is being incorrectly used as the generic identity anchor.

If the answer is not directly proven, mark it UNKNOWN.

---

## 10. Q2 Active-Gate Investigation

Verify actual implementation of:

```text
email_confirmed
verification_status
trusted_role
```

Determine whether protected business access is actually blocked when:

- email is unconfirmed;
- identity is PENDING;
- identity is REJECTED;
- trusted_role is NULL;
- an already-valid session becomes non-active.

Do not accept UI-only restrictions as proof.

Verify server-side enforcement.

---

## 11. Q3 Session / Middleware Investigation

Inspect the actual Middleware/session architecture.

Verify:

- whether Middleware exists;
- whether it uses the supported Supabase SSR mechanism;
- whether `getUser()` is used appropriately for server verification;
- which routes are covered;
- whether protected API routes are covered;
- whether session refresh occurs correctly;
- whether invalid/expired sessions fail safely;
- whether stale JWT state can preserve business access after a live Freight identity state change;
- whether cookie/Origin protections are actually implemented where required.

Do not merely say "middleware exists." Verify its effective role.

---

## 12. Q5 Rate-Limit Investigation

Verify current authentication request handling without inventing platform limits.

Determine:

- whether authentication flows rely on Supabase-native limits;
- whether a custom limiter exists;
- whether the application has accidentally added an unapproved custom limiter;
- how `429` is handled;
- whether login identifier probing creates an additional enumeration risk.

Do not reproduce high-volume abuse against production.

---

## 13. Q6 RLS / Service-Role Investigation

Inventory authentication-related `service_role` usage.

At minimum identify:

- files importing the service-role client;
- whether any public login/signup route uses it;
- whether ordinary CRUD bypasses RLS unnecessarily;
- whether privileged verification operations are separated from ordinary user operations;
- whether RLS exists for current identity tables;
- whether `freight_identities` is absent/present;
- whether protected tables have appropriate RLS.

Do not claim a complete Q6 audit unless all relevant protected tables have been considered.

---

## 14. Driver Code / Driver ID Investigation

The local flow introduces a database-generated Driver Code.

Investigate:

1. Whether the generation mechanism is a valid business-identifier feature.
2. Whether Driver Code is being treated as a secret credential.
3. Whether Driver Code is enumerable.
4. Whether public lookup of Driver Code reveals account existence.
5. Whether Driver Code → email → Supabase login creates an account-enumeration or privilege boundary problem.
6. Whether the final Node 2 design even requires Driver Code to be an authentication input.
7. Whether Driver Code should instead remain a post-verification application identifier.

Do not decide to keep/remove Driver Code based solely on UI preference.

---

## 15. Migration / Database-State Investigation

Inspect all relevant migrations in the GitHub source repository and local filesystem.

Pay particular attention to:

```text
004_auto_generate_driver_code.sql
```

Verify:

- migration contents;
- migration ordering;
- schema objects created;
- whether objects already exist elsewhere;
- compatibility with the locked Freight Identity design;
- whether the migration is local-only;
- whether it is applied in the relevant database(s), where safely verifiable;
- effect on existing rows;
- failure/rollback implications.

Do not apply the migration during this investigation.

---

## 16. Correctness / Build / Test Evidence

For source-state checks, collect safe evidence such as:

- TypeScript compile/check;
- lint if configured;
- existing automated tests;
- authentication route tests;
- route-level protected-access checks;
- migration validation that does not mutate production.

Record exact commands/results.

For production, use only safe non-mutating verification unless explicitly authorized.

---

## 17. Required Finding Classification

Every material finding must be classified as one of:

```text
VERIFIED
INFERRED
UNKNOWN
```

and placed into a practical category:

```text
Production/deployment mismatch
Local-only implementation
GitHub-only implementation
Existing bug
Node 2 requirement satisfied
Node 2 requirement missing
Security gap
Database/migration risk
Architecture conflict
Intentional but undocumented behavior
Evidence gap
```

Do not label an architecture conflict unless the locked contract itself cannot be preserved by implementation.

---

## 18. Required Final Output

Produce a concise, evidence-rich report with:

```text
1. Exact GitHub commit inspected
2. Exact Vercel deployment commit (or UNKNOWN)
3. Exact local commit/status
4. Screenshot/UI differences
5. GitHub vs local source differences
6. Vercel vs GitHub relationship
7. Authentication-flow comparison
8. Q1 findings
9. Q2 findings
10. Q3 findings
11. Q4 findings
12. Q5 findings
13. Q6 findings
14. Driver ID / Driver Code findings
15. Database/migration findings
16. Build/test evidence
17. Finding classification table
18. What is VERIFIED
19. What remains UNKNOWN
20. Correct target behavior under locked Node 2
21. Focused investigations still required
22. Push/discard recommendation
23. Final conclusion
```

The final conclusion must answer explicitly:

```text
Why are Vercel and localhost different?
Which code is Vercel running?
Which code is localhost running?
Which differences are intentional vs accidental?
Which differences violate Node 2?
Which behavior should be preserved?
Which behavior should be replaced?
What must NOT be pushed?
What focused investigation comes next?
```

---

## 19. Completion Gate

This investigation is complete only when the evidence is sufficient to safely derive the exact Node 2 implementation scope.

Do not create an implementation prompt merely because the application is currently working.

Do not discard local changes until the evidence and decision are recorded.

Do not push local changes during this investigation.

Final sequence:

```text
Vercel
  ↕
GitHub source
  ↕
Localhost
  ↓
Evidence reconciliation
  ↓
Compare against locked Q1–Q7
  ↓
Focused Node 2 investigations if required
  ↓
Implementation scope
  ↓
Antigravity implementation prompt
```
