# Chat37 — Node7 Phase1a — Public Share 404 — Create-Flow / DB Schema-Mismatch Fix Prompt

## 1. Purpose

Implement the verified fix for the production Public Share 404 discovered in:

`05_DEBUGGING/investigations/Chat37_Node7_Phase1a_PublicShare404_CreateFlow_DB_Reconciliation_Report.md`

This is a localized source-code fix. Do not redesign the Public Share architecture and do not modify the database schema.

## 2. Verified Root Cause

The Public Share create flow is working correctly:

- secure token generation is correct
- SHA-256 hashing is correct
- `trip_public_shares` persistence is correct
- `ACTIVE` status is correct
- returned public token/URL is correct
- public verification receives the token
- the stored token hash is found correctly

The failure occurs inside:

`src/lib/public-share-lookup.ts`

The `trips` query requests columns that do not exist in the production `trips` schema:

- incorrect: `pickup_city`
- incorrect: `destination_city`

The verified production schema uses:

- `facility_name`
- `destination_name`

Supabase therefore returns PostgreSQL error `42703` (column does not exist). The helper currently masks this lookup failure as `null`, which causes the public API/page to return a generic 404.

## 3. Authorized Change

Modify only the public-share lookup/projection code necessary to align the trip query with the verified production schema.

### Required

In:

`src/lib/public-share-lookup.ts`

1. Replace the non-existent trip columns `pickup_city` and `destination_city` with the verified existing columns `facility_name` and `destination_name`.
2. Preserve the public projection semantics so the public verification page continues to receive the correct pickup/facility and destination information.
3. Keep the existing token hashing and `trip_public_shares` lookup unchanged.
4. Keep `ACTIVE` status enforcement unchanged.
5. Keep rate limiting in the API route unchanged.
6. Keep evidence/event filtering unchanged unless compilation requires a strictly local type/property adjustment caused by the column-name correction.
7. Do not change database schema, migrations, token format, token persistence, revocation behavior, or rate-limit behavior.
8. Do not perform unrelated cleanup/refactoring.

## 4. Error-Handling Requirement

Do not introduce broader error swallowing.

The existing investigation established that the schema error was being converted into `null` and therefore hidden behind a 404. The implementation should not make this class of database failure harder to diagnose.

If a strictly local error-handling adjustment is required to preserve correct behavior, unexpected database/query failures should remain distinguishable from a genuinely missing/inactive public share.

Do not redesign the error model beyond what is necessary for this localized fix.

## 5. Files / Scope

Primary expected file:

`src/lib/public-share-lookup.ts`

Only modify another source file if the direct schema-name correction creates a concrete compile/type issue that must be resolved for the same fix. If additional changes are required, document the exact reason in the implementation report.

Do not modify:

- database schema
- database records
- public-share token generation
- public-share creation endpoint behavior
- authentication/authorization
- rate limiting
- unrelated UI
- unrelated trip logic
- unrelated Node7 work

## 6. Validation

After implementation:

1. Run TypeScript validation.
2. Run the production build.
3. Verify the public-share helper query now uses `facility_name` and `destination_name`.
4. Verify no accidental references to `pickup_city` or `destination_city` remain in the affected public-share lookup path.
5. If safe and available locally, perform a focused test of the public-share lookup using a valid test share. Do not fabricate production database evidence.
6. Do not claim production success unless the deployed production URL is actually tested.

## 7. Production Verification Handoff

After the source fix is implemented and locally validated, stop and provide the implementation report.

Do NOT push or deploy unless Ayush separately authorizes the push/deployment action.

After deployment authorization, Ayush will manually verify a newly generated production Public Share URL.

Expected successful flow:

`Create Share → ACTIVE DB row → token hash match → trip lookup succeeds → projection populated → Public API 200 → Public Share page renders`

## 8. Required Implementation Report

Create/update:

`03_IMPLEMENTATION/implementation_reports/Chat37_Node7_Phase1a_PublicShare404_CreateFlow_DB_SchemaMismatchFix_Implementation_Report.md`

The report must include:

- status
- exact files changed
- exact schema mismatch fixed
- confirmation that token generation/persistence was not changed
- validation commands and results
- whether production was tested
- any concerns or remaining UNKNOWN items
- explicit statement that no DB/schema/deployment changes were made by this implementation unless separately authorized

## 9. Stop Condition

Stop after the localized fix and validation/report are complete.

Do not proceed into Phase1b UI redesign or Phase3 AI work as part of this task.
