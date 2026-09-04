# Chat37 — Node7 Phase1a — Public Share 404 — DB + Code Reconciliation Investigation Prompt

## Role
You are the implementation/execution agent (Antigravity). This is an **INVESTIGATION ONLY** task.

Do not modify source code, database data/schema, environment variables, deployment configuration, or production deployment.
Do not regenerate/revoke tokens.
Do not push commits.

## Objective
Determine the exact production failure point causing the public share URL to return a generic 404 after deployment of commit `8d90977` (`Refactor public share lookup architecture`).

The investigation must verify **both database state and code/runtime path**. Do not guess the root cause.

## Exact Production URL
Ayush manually observed:
`https://freighthackathon.vercel.app/share/VPV9OUu7VwVqRCeeJUWROYA1H76IPRa9ioRTRVaNc`

Observed browser result: generic Next.js 404.

Treat the token as sensitive operational data: do not place the raw token or derived token hash in any report.

## Investigation Procedure

### 1. Verify deployed source
Confirm Production is running commit `8d90977`.
Inspect the deployed/source versions of:
- `src/lib/public-share-lookup.ts`
- `src/app/api/public/verify/[token]/route.ts`
- `src/app/share/[token]/page.tsx`

Confirm whether the deployed code matches the current `main` source.

### 2. Trace the token algorithm
Inspect the application's existing `hashToken` implementation and derive the exact lookup hash locally/in memory for investigation only.
Never record the raw token or hash in the report.

### 3. Verify production database state — READ ONLY
Using the production Supabase project/database, verify the exact share lookup:
- Does the matching `trip_public_shares` row exist?
- Is its status `ACTIVE`?
- Does it contain a valid `trip_id`?
- Does the referenced `trips` row exist?
- Does the referenced receiving company exist?
- Do the related `events` rows exist?
- What relevant event types/timestamps/location fields are present?

Record safe evidence only. No writes.

### 4. Verify API independently
Request:
`https://freighthackathon.vercel.app/api/public/verify/<exact-token>`

Do not record the raw token in the report.

Record:
- HTTP status;
- whether a public projection or error response was returned;
- safe relevant error/log evidence.

Interpretation must remain evidence-based:
- 200 → helper/database path likely succeeds; continue tracing page.
- 404 → determine whether helper returned null because of database lookup/data state.
- 500 → investigate actual server-side exception/log.

### 5. Trace page execution
Determine whether production `/share/[token]`:
- resolves the dynamic route;
- receives the token through the Next.js 16 Promise params contract;
- invokes `getPublicVerificationData(token)`;
- receives a projection, `null`, or thrown exception;
- reaches `notFound()`;
- fails before helper execution.

Distinguish a route-level 404 from a helper/data-level 404.

### 6. Inspect error masking
The current helper may return `null` for database/query failures. Verify whether any actual Supabase/query/runtime failure is being converted into `null`, then into `notFound()` and browser 404.

Do not treat a 404 alone as proof that the token is invalid.

### 7. Reconcile all evidence
Use this structure in the report:

| Stage | Expected | Actual | Confidence |
|---|---|---|---|
| Production commit | `8d90977` | | |
| Dynamic route resolution | token resolved | | |
| Token hash | app algorithm | | |
| `trip_public_shares` | ACTIVE row | | |
| `trip_id` | valid | | |
| Trip lookup | valid | | |
| Company lookup | valid/optional | | |
| Events lookup | executes | | |
| Helper result | projection | | |
| API response | 200 projection | | |
| Page render | verification page | | |

Use only these confidence labels:
- VERIFIED
- INFERRED
- UNKNOWN

## Security Constraints
- No source changes.
- No database writes.
- No schema changes.
- No RLS/policy changes.
- No environment changes.
- No deployment.
- No token regeneration/revocation.
- Never record raw tokens, token hashes, API keys, service-role keys, cookies, auth credentials, or secret environment values.

## Required Report
Create/update exactly:
`05_DEBUGGING/investigations/Chat37_Node7_Phase1a_PublicShare404_DB_Code_Reconciliation_Report.md`

The report must contain:
1. Observation.
2. Scope.
3. Code evidence.
4. Database evidence.
5. API evidence.
6. Page/runtime evidence.
7. Expected-vs-actual reconciliation.
8. Root cause only if directly supported; otherwise `UNKNOWN`.
9. Confidence tags.
10. Recommended next decision.
11. Explicit statement that no source/database/deployment changes were made.

## Completion Gate
Stop after the investigation report is complete. Do not implement a fix even if a likely cause is found. The next implementation decision will be made separately by the reasoning brain after reviewing your evidence.
