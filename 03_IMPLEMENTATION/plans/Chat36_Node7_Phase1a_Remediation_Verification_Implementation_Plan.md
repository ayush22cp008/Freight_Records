# Remediation & Verification Plan (Chat36)

Based on the inspection of the source code against the locked Phase 1a architecture, I have formulated the following remediation plan.

## Proposed Changes

### AI Grounding & Freshness
- **Finding**: The architecture lacks a dedicated AI storage/caching table (e.g., `trip_summaries`). Executing a live LLM call on every anonymous `GET /api/public/verify/[token]` hit would expose a severe Denial-of-Wallet vulnerability since there is no cache to bind to an evidence fingerprint.
- **Action**: 
  - [MODIFY] `src/app/api/public/verify/[token]/route.ts`: Remove the mock fetch to `trip_summaries`.
  - [MODIFY] `src/app/api/public/verify/[token]/route.ts`: Hardcode the fallback `"AI summary unavailable."` gracefully to preserve Timeline integrity and avoid inventing a new caching architecture.
  - Report the lack of an AI caching architecture as a persistent UNKNOWN that blocks live public AI generation.

### Audit
- **Finding**: A thorough search of migrations and `src/lib/` reveals no existing authoritative `audit` table or mechanism from Node 6.
- **Action**:
  - Stop and report the exact UNKNOWN: "No authoritative audit architecture exists in the repository. Implementing one would require inventing new schemas and conventions, violating the restriction against creating parallel audit systems."

### Concurrency
- **Finding**: Concurrency safety of the partial unique index (`unique_active_share`) was INFERRED in Chat35, but not tested.
- **Action**:
  - Write a small test script (`tests/concurrency.ts`) that executes 10 concurrent `POST /api/trips/[tripId]/public-share` requests using `Promise.all()`.
  - Verify that exactly 1 ACTIVE share exists after the race condition.

## Open Questions & Review Required
> [!WARNING]
> Since there is no existing caching mechanism for AI Summaries, running expensive LLM calls on a public endpoint is unsafe. Is it acceptable to hard-code the AI summary as "unavailable" until an AI caching table is introduced in a future node?
> Additionally, since no audit table exists, we will not create one to avoid inventing schemas. Is this acceptable?
