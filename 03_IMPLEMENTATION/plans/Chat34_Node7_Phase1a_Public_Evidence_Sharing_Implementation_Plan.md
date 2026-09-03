# Node 7 Phase 1a Public Evidence Sharing Implementation Plan

This plan maps the requirements from `Chat34_Node7_Phase1a_Public_Evidence_Sharing_Antigravity_Implementation_Prompt.md` into concrete code changes within the `freight_hackathon` repository.

## Proposed Changes

### Database Migration

#### [NEW] `src/db/migrations/006_create_trip_public_shares.sql`
- Create `trip_public_shares` table with `id`, `trip_id`, `token_hash`, `status`, `created_by`, `created_at`, `revoked_at`.
- Add foreign key constraints to `trips` (`CASCADE`) and `freight_identities` / `auth.users` (`SET NULL`).
- Add partial unique index: `CREATE UNIQUE INDEX unique_active_share ON trip_public_shares(trip_id) WHERE status = 'ACTIVE';`

### Share Lifecycle API (Company Portal)

#### [NEW] `src/app/api/trips/[tripId]/public-share/route.ts`
- **POST**: Handles creating and replacing a public share.
  - Verifies `COMPANY` role and trip ownership using `receiving_company_id`.
  - Ensures the trip is `completed` and has `DELIVERY_DEPARTED` recorded (using existing rules).
  - Uses a single transaction to revoke the old active share (if any) and insert the new active share.
  - Generates a 256-bit random token (32 bytes), base64url encodes it, hashes it with SHA-256 for the database, and returns the raw token in a `publicUrl`.
- **DELETE**: Handles revoking a share.
  - Marks the active share as `REVOKED` and sets `revoked_at`.

### Public Verification API

#### [NEW] `src/app/api/public/verify/[token]/route.ts`
- **GET**: Anonymous public endpoint.
  - Generates SHA-256 hash of the incoming token to lookup `trip_public_shares`.
  - Enforces basic rate limiting by IP (using in-memory map or simple KV depending on Node 6 rate limit availability).
  - Verifies status is `ACTIVE`.
  - Resolves trip data, company info, and chronological events (Arrival, Check-in, Departure).
  - Omits private fields (driver info, internal IDs, raw photos).
  - Invokes AI summary (if available) bound to the current evidence state.

### Public UI Page

#### [NEW] `src/app/share/[token]/page.tsx`
- A strictly read-only Server Component.
- Applies `export const dynamic = 'force-dynamic'` and no-index metadata.
- Fetches from the `/api/public/verify/[token]` route (or executes the logic server-side directly).
- Displays Company Name, Delivery Date, Pickup/Destination City, Evidence Checklist, Timeline, and AI Summary.

### Security & Auditing

#### [NEW] `src/lib/audit.ts` (if none exists)
- Create a lightweight audit logger `logAudit(action, trip_id, user_id, details)` to track `share_created`, `share_accessed`, `share_revoked`, `share_replaced`.

## Verification Plan

### Automated Tests
- If applicable, execute existing API tests or create a quick test script (`tests/share.test.js`) to verify token generation, partial unique index constraints, and public data exposure.

### Manual Verification
- A verification script will be provided for Ayush to test Company portal generation, anonymous public access, revocation, and replacement, ensuring raw photos and driver identities are perfectly hidden.
