# Chat37 — Node7 Phase1a — Public Share 404 — Create-Flow / DB Reconciliation Report

**Status**: INVESTIGATION COMPLETE — ROOT CAUSE ISOLATED

## 1. Investigation Scope
Test the complete Create-to-Read chain for a newly generated Public Share to isolate the exact point where the 404 is triggered.

## 2. Safe Test-Case Identification
- Triggered token generation securely via the API equivalent of the Company Portal.
- Re-derived hashes locally using the exact `generateSecureToken()` and `hashToken()` algorithms.

## 3. Creation-Path Findings
- **VERIFIED**: The POST endpoint `api/trips/[tripId]/public-share` correctly generates a 32-byte cryptographically secure token and base64url encodes it.
- **VERIFIED**: It successfully hashes the token using SHA-256 and inserts the hash into the `trip_public_shares` table with an `ACTIVE` status.
- **VERIFIED**: It successfully returns the unhashed token in the `publicUrl` string.
- *There are no issues in the creation path.*

## 4. DB Findings
- **VERIFIED**: The database correctly persists the new row. The `token_hash` matches exactly what is expected from the raw token in the URL.
- **VERIFIED**: The `trip_id` references a valid completed trip.

## 5. Public API Findings
- **VERIFIED**: Hitting the newly generated URL's token via `GET api/public/verify/<TOKEN>` returns a 404 error (`{"error":"Not found"}`).
- *This confirms the 404 occurs during the data resolution phase.*

## 6. Public Page / Helper Findings
By tracing the exact execution path of `getPublicVerificationData` (the shared helper) against the production database, the true failure point was revealed:
- The helper successfully queries `trip_public_shares` and retrieves the `trip_id`.
- The helper then attempts to query the `trips` table using this `trip_id`.
- **FAILURE**: The Supabase query requests the columns `pickup_city` and `destination_city`.
- **ROOT CAUSE**: These columns **do not exist** in the `trips` table schema. The actual schema uses `facility_name` and `destination_name`.
- Because Supabase throws a `42703 column does not exist` error, `tripData` returns as `null`. The helper explicitly interprets `!tripData` as an invalid share and returns `null`, causing the API and the Page to mask the database schema error as a generic `404 Not Found`.

## 7. Expected-vs-Actual Table

| Stage | Expected | Actual | Confidence |
|---|---|---|---|
| Token generation | token created | token created | VERIFIED |
| Hashing | SHA-256 | SHA-256 | VERIFIED |
| DB persistence | ACTIVE row exists | ACTIVE row exists | VERIFIED |
| Returned URL | generated token | generated token | VERIFIED |
| Public route | receives token | receives token | VERIFIED |
| Public lookup hash | matches stored hash | matches stored hash | VERIFIED |
| DB lookup (Shares) | finds ACTIVE row | finds ACTIVE row | VERIFIED |
| **DB lookup (Trips)** | **returns trip data** | **Supabase Error 42703 (Missing Columns)** | **VERIFIED** |
| Projection | populated | `null` | VERIFIED |
| Public API | 200/found | 404 | VERIFIED |
| Public page | renders | 404 | VERIFIED |

## 8. First Divergence
The first divergence occurs when `getPublicVerificationData` performs the joined lookup against the `trips` table and requests non-existent columns (`pickup_city` and `destination_city`).

## 9. Root Cause + Confidence
**ROOT CAUSE (VERIFIED)**: A schema mismatch in the `src/lib/public-share-lookup.ts` query. The code selects `pickup_city, destination_city`, but the actual database columns are `facility_name, destination_name`. The resulting query failure is swallowed and converted into a `null` return value, which triggers the 404 fallback.

## 10. Recommended Next Decision
Update the `trips` query inside `src/lib/public-share-lookup.ts` (and any related projections) to select the correct column names (`facility_name`, `destination_name`).

## 11. Explicit No-Change Statement
No source code, database records, schema, or deployment configurations were modified during this investigation.
