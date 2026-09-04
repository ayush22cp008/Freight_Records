# Chat37 — Node7 Phase1a Public Share 404 — Create-Flow / DB Reconciliation Investigation Prompt

## Role

You are Antigravity, the implementation/investigation execution agent.

This task is **INVESTIGATION ONLY**. Do not modify source code, database data/schema, environment configuration, or deployment state.

## Read First

Primary investigation record:

`05_DEBUGGING/investigations/Chat37_Node7_Phase1a_PublicShare404_CreateFlow_DB_Reconciliation_Investigation.md`

Relevant prior investigation/report:

`05_DEBUGGING/investigations/Chat37_Node7_Node7_Phase1a_PublicShare404_DB_Code_Reconciliation_Report.md`

If the second path is unavailable, use the Records repository history/search to locate the existing Chat37 Phase1a DB/code reconciliation report; do not invent a replacement conclusion.

## Current Production Observation

A brand-new Public Share was generated from the production Company Portal.

The UI displayed:
- `Share generated successfully!`
- `Status: Active`
- a new `/share/<token>` URL

Opening that newly generated URL still returns the generic Next.js 404 page.

This means the investigation must now test the complete create-to-read chain. Do not assume the token is nonexistent merely because the public page returns 404.

## Locked Context

- Chat: Chat37
- Hackathon Day: Day14
- Node: Node7
- Phase: Phase1a
- Production app: `https://freighthackathon.vercel.app`
- Source branch: `main`
- Relevant deployed commit: `8d909776bb4ad947d85f0ce08d00bd3407b31d92`

## Investigation Objective

Find the first divergence in:

`Create Public Share → token generation → hash → DB persistence → returned URL → public route → hash → DB lookup → projection → API/page`

## Required Work

### A. Inspect creation path

Identify the exact source endpoint/component/action invoked by the production Company Portal `Create Public Share` button.

Determine:
- token generation mechanism
- token encoding/length if relevant
- hash algorithm
- exact DB table/columns written
- status written
- trip/company/user linkage
- whether insert/update is awaited
- error handling
- whether success can be shown if persistence fails
- exact token returned to the client/UI

### B. Inspect public read path

At deployed commit `8d909776bb4ad947d85f0ce08d00bd3407b31d92`, inspect:

- `src/app/share/[token]/page.tsx`
- `src/app/api/public/verify/[token]/route.ts`
- `src/lib/public-share-lookup.ts`
- the actual share-creation source path

Confirm the public route receives the dynamic token, hashes it with the same algorithm, and queries `trip_public_shares` with the expected conditions.

### C. Production DB verification

Using the production DB connection already available to the application, perform **read-only** verification for the newly generated share.

You may derive the token hash locally using the exact application algorithm, but:

**DO NOT record the raw token or SHA-256 hash in any Records file, terminal output copied into the report, or implementation report.**

Verify:
- matching `trip_public_shares` row exists
- status is `ACTIVE`
- trip_id is valid
- created/updated timing is consistent with the manual generation
- referenced trip exists
- related company data exists/readable
- no duplicate/conflicting row explains the behavior

### D. Verify production API

Call the production API for the newly generated share:

`https://freighthackathon.vercel.app/api/public/verify/<NEW_TOKEN>`

Do not put the token in the report.

Record only:
- HTTP status
- found/not-found/server-error classification
- whether it agrees with the DB observation

### E. Reconcile the chain

Create an evidence table:

| Stage | Expected | Actual | Confidence |
|---|---|---|---|
| Token generation | token created | | |
| Hashing | SHA-256 | | |
| DB persistence | ACTIVE row exists | | |
| Returned URL | generated token | | |
| Public route | receives token | | |
| Public lookup hash | matches stored hash | | |
| DB lookup | finds ACTIVE row | | |
| Projection | populated | | |
| Public API | 200/found | | |
| Public page | renders | | |

Use `VERIFIED`, `INFERRED`, or `UNKNOWN` only.

## Root-Cause Rule

Do not declare a root cause until the first divergence is supported by evidence.

If production DB access is blocked, do not guess. Mark the exact DB fact UNKNOWN and report the concrete blocker and the smallest next test required.

## No-Change Boundary

Do NOT:
- edit source code
- edit/delete/insert DB rows
- change schema
- change environment variables
- change Vercel configuration
- redeploy
- refactor
- clean up unrelated code

Read-only inspection and API verification only.

## Required Report

Create/update exactly:

`05_DEBUGGING/investigations/Chat37_Node7_Phase1a_PublicShare404_CreateFlow_DB_Reconciliation_Report.md`

The report must include:
1. investigation scope
2. safe test-case identification
3. creation-path findings
4. DB findings
5. public API findings
6. public page/helper findings
7. expected-vs-actual table
8. first divergence
9. root cause + confidence
10. recommended next decision
11. explicit no-change statement

Do not put secrets, raw token values, token hashes, Supabase credentials, or environment secrets in the report.

## Completion Condition

Stop after the investigation report is complete. Do not implement a fix.

The next action will be decided by the reasoning brain after reviewing the report.
