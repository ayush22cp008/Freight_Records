# Antigravity Remediation & Verification Prompt — Chat35 Node 7 Phase 1a

Continue the Node 7 Phase 1a work from the existing Chat35 implementation.

This is a remediation and verification pass only. Do not begin Phase 1b.

## AUTHORITATIVE SOURCES

Primary:
`03_IMPLEMENTATION/plans/Chat35_Node7_Phase1a_Public_Evidence_Sharing_Reconciled_Implementation_Plan.md`

Supporting locked sources:
- `03_IMPLEMENTATION/prompts/Chat34_Node7_Phase1a_Public_Evidence_Sharing_Antigravity_Implementation_Prompt.md`
- `02_ARCHITECTURE/Chat34_Node7_Phase1a_Public_Evidence_Sharing_Architecture_Finalization.md`
- `00_PROJECT_CONTROL/CHECKPOINTS/Chat30_Node7_Phase1a_Public_Evidence_Sharing_Decision_Checkpoint.md`
- `05_DEBUGGING/investigations/Chat32_Node7_Phase1a_Remaining_Source_Reconciliation_Investigation.md`

Existing report:
`03_IMPLEMENTATION/implementation_reports/Chat35_Node7_Phase1a_Public_Evidence_Sharing_Implementation_Report.md`

## CRITICAL PROTECTION RULE

DO NOT MODIFY, REWRITE, DELETE, RENAME, OR OVERWRITE the existing Chat34 implementation plan:
`03_IMPLEMENTATION/plans/Chat34_Node7_Phase1a_Public_Evidence_Sharing_Implementation_Plan.md`

Do not reopen Nodes 1–6. Do not change the locked Phase 1a architecture. Do not begin Phase 1b.

## 1. INSPECT FIRST

Inspect the current implementation and repository state before changing anything, especially:
- `src/app/api/public/verify/[token]/route.ts`
- `src/app/api/trips/[tripId]/public-share/route.ts`
- `src/lib/public-share.ts`
- `src/db/migrations/006_create_trip_public_shares.sql`
- existing AI summary implementation
- evidence source of truth
- existing audit conventions
- Node 6 rate-limiting conventions
- existing tests

The Chat35 report identified three unresolved areas: AI grounding UNKNOWN, audit UNKNOWN, and concurrency only INFERRED. Verify these against actual source rather than assuming the report is correct.

If implementation reality conflicts with locked architecture/Chat35, STOP and report the exact discrepancy. Do not silently invent a replacement.

## 2. AI GROUNDING AND FRESHNESS

Remove any production mock such as `trip_summaries` from the public verification path if present. Use the existing authoritative AI/evidence flow and do not create a second evidence source of truth.

Verify that Arrival + Check-in + Departure requirements remain aligned with the established summary flow and that the public summary is grounded in current authoritative evidence.

Verify freshness: stale AI output must not be served as current. Use an existing fingerprint/version mechanism if present. If the locked architecture does not provide enough information for a safe implementation, STOP and report rather than inventing a new AI storage architecture.

If AI is unavailable for a valid share: return HTTP 200, deterministic evidence/timeline data, and a generic unavailable state without provider/model/error details.

## 3. AUDIT

Inspect existing Node 6/project audit conventions. Required Phase 1a events are:
- share created
- share accessed
- share revoked
- share replaced

Use the existing authoritative audit mechanism if one exists. Do not create a parallel audit system. If no mechanism exists, implement only what the locked architecture specifies; if it is insufficient to determine the correct implementation, STOP and report the UNKNOWN.

Never persist, log, or audit raw bearer tokens. Never expose audit information publicly.

## 4. CONCURRENCY

Convert the current INFERRED concurrency claim to VERIFIED only through actual testing.

Test concurrent public-share creation/replacement and verify that at most one ACTIVE share exists per trip. Verify transaction/locking behavior, eligibility re-checks, revoke/insert sequence, partial unique index enforcement, and defensive unique-violation handling.

Do not merely state that the index theoretically guarantees correctness.

## 5. ANONYMOUS RATE LIMITING

Inspect and test `checkAnonymousRateLimit()` and the public verification route.

Verify:
- rate limiting occurs before expensive verification/AI work
- IP + opaque supplied-token composite protection
- IP-only protection for malformed/invalid requests
- raw bearer tokens are never persisted
- behavior follows the locked Phase 1a architecture and Node 6 conventions

Run actual tests. Do not mark VERIFIED solely because the function exists. If an exact threshold/mechanism is genuinely unsupported by locked sources, STOP and report rather than inventing policy.

## 6. PRIVACY / TOKEN / PUBLIC PAGE

Verify the actual API response and page do not expose driver identity, raw photos/photo URLs, exact GPS/GPS accuracy, street/full addresses, internal IDs, private customer/company details, vehicle identifiers/registration, AI provider/model details, or audit/security details.

Verify token lifecycle: 32 cryptographically secure random bytes, Base64URL token, SHA-256 hash persisted only, raw token never persisted/logged/audited, replacement invalidates old token, revocation invalidates token, and invalid/malformed/revoked/nonexistent tokens have generic 404 behavior.

Verify the public page is dynamic, no-store/no-cache as required, noindex, deployment-aware, read-only, anonymous, and has no management/download/PDF/export/direct-DB functionality.

## 7. FULL VERIFICATION

Run the complete relevant Phase 1a verification suite, including:
- database/migration and constraints
- authorization and company→trip boundary
- token lifecycle/security
- public API behavior
- privacy
- evidence/timeline
- AI success/failure/freshness
- audit
- concurrency
- anonymous rate limiting
- cache/noindex behavior
- TypeScript/typecheck
- lint if configured
- build
- relevant existing tests

Record the exact commands and actual results. Do not claim VERIFIED without execution evidence.

## 8. UPDATE THE CHAT35 REPORT

Update:
`03_IMPLEMENTATION/implementation_reports/Chat35_Node7_Phase1a_Public_Evidence_Sharing_Implementation_Report.md`

Accurately distinguish VERIFIED, INFERRED, and UNKNOWN. Specifically resolve the previously reported AI grounding UNKNOWN, audit UNKNOWN, and concurrency INFERRED. If any remain unresolved, document exactly why.

Include exact files changed, migration/index, APIs, authorization, token security, concurrency evidence, AI source/freshness/failure behavior, audit behavior, rate-limit evidence, privacy verification, cache/indexing verification, exact tests/commands, build/typecheck/lint results, remaining UNKNOWNs, manual verification checklist, and final commit SHA.

## 9. COMMIT AND STOP

After remediation and verification:
1. Commit the Phase 1a changes.
2. Push the commit.
3. Record the final commit SHA in the report.
4. Report exactly what changed and what was verified.
5. Report any remaining UNKNOWN items.
6. State clearly that the implementation is READY FOR AYUSH MANUAL VERIFICATION.

Then STOP. Do not start Phase 1b, redesign the architecture, add Phase 3 features, or perform unrelated refactoring.

## FINAL RULE

Inspect → remediate only identified Phase 1a gaps → run actual verification → update Chat35 report → commit/push → STOP for Ayush manual verification.
