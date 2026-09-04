# Chat37 — Node7 Phase1a — Public Share 404 — Shared Lookup Architecture Fix

## Authorization
IMPLEMENTATION AUTHORIZED by Ayush after investigation review.

This prompt authorizes a focused architecture fix for the public-share page. The goal is to remove the unnecessary Server Component → own HTTP API dependency that can convert runtime/API failures into an indistinguishable generic 404.

## Evidence Basis
The completed investigation established:

- The Next.js 16 dynamic `params` issue was fixed and deployed in commit `a71a66f`.
- `src/app/share/[token]/page.tsx` correctly awaits `params`.
- The page performs a server-side HTTP fetch to `/api/public/verify/[token]` using `NEXT_PUBLIC_APP_URL`.
- Fetch exceptions and all non-OK responses are converted to `null`.
- `null` is converted to `notFound()`, masking network/API failures as a generic 404.
- The actual production runtime/configuration failure remains unknown.

Therefore the authorized fix is to remove this Server Component's dependency on an absolute self-HTTP request rather than guessing at Vercel environment configuration.

## Required Architecture
Create a small shared server-side helper containing the reusable public-share verification/data lookup logic.

The intended structure is:

`src/lib/public-share-lookup.ts`

The helper must encapsulate the database-backed lookup/projection logic needed by the public share page and the verification API.

Then:

- `src/app/share/[token]/page.tsx` calls the shared helper directly.
- `src/app/api/public/verify/[token]/route.ts` calls the same shared helper and remains the public HTTP API boundary.

The page must no longer construct or fetch:

`NEXT_PUBLIC_APP_URL + /api/public/verify/[token]`

## Functional Requirements

### Shared helper
The helper must preserve the current verification semantics:

1. Hash the supplied public-share token using the existing `hashToken` implementation.
2. Look up an `ACTIVE` record in `trip_public_shares` using the hashed token.
3. Resolve the associated trip.
4. Resolve the receiving company name.
5. Fetch chronological events for the trip.
6. Preserve the current key-event filtering and evidence completeness semantics.
7. Preserve the current delivery-date derivation.
8. Preserve the current AI summary behavior and graceful AI-summary fallback.
9. Return the same strict public projection currently returned by the API.
10. Do not expose private/internal fields.

### API route
Preserve the existing API contract as closely as possible:

- Keep anonymous rate limiting in the API route boundary.
- Keep token validation in the API route boundary.
- Use the shared helper for the verification/data resolution.
- Preserve existing HTTP status semantics for invalid/nonexistent/revoked shares.
- Preserve no-cache response headers.
- Preserve the existing 500 handling for unexpected server errors.

Do not move API-specific concerns such as request headers/rate limiting into the generic data helper.

### Public share page
The page must:

- Continue using `params: Promise<{ token: string }>` and `await params`.
- Call the shared server-side helper directly.
- No longer use `NEXT_PUBLIC_APP_URL`.
- No longer make an HTTP request to its own API route.
- Preserve the existing public verification UI and projection fields.
- Continue rendering `notFound()` for an invalid/nonexistent/revoked share according to the helper's result.

## Scope Control
This is a localized architecture refactor for the public-share verification path only.

DO NOT:
- modify unrelated pages/components;
- redesign the UI;
- change database schema;
- change public-share token generation;
- change token hashing;
- change share replacement/revocation behavior;
- change rate-limit policy;
- change authentication/authorization behavior elsewhere;
- change Nodes 1–6;
- begin Phase1b;
- begin Phase3;
- introduce a second evidence source;
- add unrelated cleanup/refactors.

## Important Implementation Constraint
Avoid duplicating the database/projection logic between the page and API route.

The shared helper should be the single source of truth for the public-share data lookup/projection. Keep HTTP-specific behavior in the API route and page-rendering behavior in the page.

Do not change database queries beyond what is necessary to extract the existing logic into the helper.

## Error Handling
The page must distinguish a legitimate missing/invalid share from unexpected server failures as much as the current architecture permits.

Do not recreate the previous behavior where arbitrary internal fetch/network failures are silently translated into a missing page solely because of an HTTP self-fetch.

For unexpected helper/database failures, use appropriate server-side error handling rather than pretending the share does not exist.

Do not expose sensitive database/error details to the public page.

## Validation Requirements
After implementation:

1. Run the project's relevant type/build validation.
2. Confirm `npm run build` passes if that is the project's existing build command.
3. Confirm both the API route and page compile against the shared helper.
4. Confirm no remaining `NEXT_PUBLIC_APP_URL` dependency exists in the public share page path.
5. Confirm no self-HTTP fetch remains in `src/app/share/[token]/page.tsx`.
6. Confirm the API's existing rate limiting remains in place.
7. Confirm token hashing and `ACTIVE` share lookup remain intact.
8. Confirm the public projection fields remain unchanged.
9. If tests exist for this area, run the relevant tests.
10. Do not claim production behavior is fixed until Ayush manually verifies the deployed result.

## Git / Deployment Boundary
Do NOT push or deploy this implementation unless Ayush separately authorizes the push/deployment step after reviewing the implementation report and validation results.

## Required Implementation Report
Write/update:

`03_IMPLEMENTATION/implementation_reports/Chat37_Node7_Phase1a_PublicShare404_SharedLookup_ArchitectureFix_Implementation_Report.md`

The report must include:

- exact files changed;
- concise architecture change;
- confirmation that page self-fetch was removed;
- confirmation that `NEXT_PUBLIC_APP_URL` is no longer required by the page path;
- confirmation that API behavior/rate limiting/token hashing/ACTIVE lookup were preserved;
- build/type/test results;
- any warnings or remaining concerns;
- explicit statement that no push/deployment was performed unless separately authorized.

Stop after implementation and validation. Do not perform production deployment without separate authorization.
