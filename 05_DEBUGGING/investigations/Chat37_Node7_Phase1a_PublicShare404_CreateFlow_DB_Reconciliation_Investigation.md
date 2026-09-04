# Chat37 — Node7 Phase1a Public Share 404 — Create-Flow / DB Reconciliation Investigation

## Status
INVESTIGATION AUTHORIZED — AWAITING ANTIGRAVITY EXECUTION

## Context

A previous investigation established that the production public-share page and API correctly execute the shared public-share lookup path, and that an older tested token was not present in the production `trip_public_shares` table. A new production Public Share was then generated from the Company Portal.

The UI reports:
- `Share generated successfully!`
- `Status: Active`
- a newly generated `/share/<token>` URL

Opening that newly generated URL still produces the generic Next.js 404 page.

This invalidates the earlier hypothesis that the failure was only caused by using an old/nonexistent token. The next investigation must reconcile the complete create-to-read path using the newly generated share as the exact test case.

## Locked Identifiers

- Chat: Chat37
- Hackathon Day: Day14
- Node: Node7
- Phase: Phase1a
- Environment: Production (`https://freighthackathon.vercel.app`)
- Source branch: `main`
- Relevant deployed architecture commit: `8d909776bb4ad947d85f0ce08d00bd3407b31d92`

## Investigation Objective

Determine exactly why a newly generated production Public Share is shown as Active by the Company Portal but the corresponding public `/share/[token]` page returns 404.

The investigation must trace and reconcile:

`Create Public Share → token generation → token hash → DB insert/update → returned URL → public route token → token hash → DB lookup → projection → page`

Do not make code, database, schema, configuration, or deployment changes during this investigation.

## Exact Evidence Required

### 1. Capture the newly generated share test case

Use the newly generated production share shown in the current manual test.

Record only safe identifiers needed for correlation. Do NOT store the raw share token or its SHA-256 hash in the Records repository.

### 2. Inspect the share creation implementation

Identify the exact production source path/function used by the Company Portal `Create Public Share` action.

Verify:
- where the random token is generated
- token length/encoding if relevant
- exact hash algorithm and encoding
- exact DB table and columns written
- exact status written
- exact trip/company/user relationship used
- whether an existing share is replaced/revoked before creation
- whether the URL token returned to the UI is exactly the generated token
- whether the insert/update is awaited and whether errors are handled
- whether the UI can display success despite a persistence failure

### 3. Reproduce the token/hash relationship safely

Using the exact application hashing algorithm, derive the SHA-256 hash of the newly generated token locally for investigation purposes only.

Do not put the raw token or derived hash in the report.

Report only whether the corresponding production DB row exists and whether the hash relationship is consistent.

### 4. Query the production database

Using the production database connection available to the application, inspect `trip_public_shares` for the newly generated share.

Verify:
- matching `token_hash` exists
- `status` is `ACTIVE`
- `trip_id` is valid
- creation/update timestamp is consistent with the manual test
- no unexpected duplicate/conflicting row exists

Then verify the referenced trip and related company data exist and are readable by the same server-side DB client used by the public-share helper.

Do not expose secrets, service-role credentials, raw tokens, or hashes in the report.

### 5. Reconcile the public read path

Inspect the deployed source at commit `8d909776bb4ad947d85f0ce08d00bd3407b31d92` and verify:

- `src/app/share/[token]/page.tsx`
- `src/app/api/public/verify/[token]/route.ts`
- `src/lib/public-share-lookup.ts`
- the share creation endpoint/component/action

Confirm the exact data path from route parameter to `hashToken(token)` to `trip_public_shares` lookup.

### 6. Test the public API with the NEW share

Call the production endpoint corresponding to the newly generated token:

`https://freighthackathon.vercel.app/api/public/verify/<NEW_TOKEN>`

Do not put the raw token in the report.

Record:
- HTTP status
- response classification (`found`, `not found`, or `server error`)
- whether the behavior matches the direct DB lookup

### 7. Compare expected vs actual

Produce a reconciliation table with these stages:

| Stage | Expected | Actual | Confidence |
|---|---|---|---|
| Token generation | token created | | |
| Hashing | SHA-256 hash | | |
| DB persistence | ACTIVE row exists | | |
| Returned URL | contains generated token | | |
| Public route | receives token | | |
| Public lookup hash | matches stored hash | | |
| DB lookup | finds ACTIVE share | | |
| Projection | populated | | |
| Public API | 200/found | | |
| Public page | evidence page renders | | |

Use only `VERIFIED`, `INFERRED`, or `UNKNOWN` confidence tags.

## Root-Cause Rules

Do not declare root cause until the evidence establishes the first divergence in the create-to-read chain.

Possible outcomes include, but are not limited to:

1. Creation UI reports success but DB persistence fails.
2. Token returned to UI differs from token persisted/hashed.
3. Token is persisted in a different environment/database than production public lookup.
4. Token hash algorithm differs between creation and lookup.
5. Row exists but status/relationship prevents lookup.
6. Public API uses a different DB/client/configuration than the creation path.
7. Public lookup is correct and another downstream projection/page problem exists.
8. The apparent 404 is caused by a different route/runtime issue.

Do not choose among these without evidence.

## Required Investigation Discipline

`OBSERVATION → INVESTIGATION → EVIDENCE → ROOT CAUSE → DECISION`

This is investigation only. No FIX stage is authorized yet.

## Change Boundary

Forbidden during this investigation:
- source-code modifications
- database writes
- schema changes
- token regeneration as a code change
- environment-variable changes
- deployment changes
- unrelated cleanup/refactoring

Read-only inspection and controlled production API/database verification are allowed.

## Required Output

Create:

`05_DEBUGGING/investigations/Chat37_Node7_Phase1a_PublicShare404_CreateFlow_DB_Reconciliation_Report.md`

The report must contain:
1. exact investigation scope
2. production test-case identification without exposing token/hash
3. code-path findings
4. DB findings
5. API findings
6. expected-vs-actual reconciliation table
7. first divergence
8. root cause with confidence
9. recommended next decision
10. explicit statement that no code/DB/config/deployment changes were made

## Completion Condition

Investigation is complete only when the newly generated share has been traced from creation through production DB persistence and public lookup, or when a specific remaining UNKNOWN is explicitly identified with the exact evidence needed to resolve it.
