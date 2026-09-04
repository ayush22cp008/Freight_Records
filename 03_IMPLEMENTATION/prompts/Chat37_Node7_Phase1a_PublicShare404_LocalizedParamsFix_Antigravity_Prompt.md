# Chat37 — Node 7 Phase 1a — Antigravity Implementation Prompt

## Objective

Implement the single localized fix proven by the Chat37 code-side investigation for the production public-share `404`.

## Root Cause — VERIFIED

The public share page `src/app/share/[token]/page.tsx` uses the pre-Next.js-16 synchronous dynamic-route `params` contract while the project is on Next.js `16.3.1`.

Because `params` is a Promise, the current page accesses `params.token` synchronously, producing `undefined`. The page then requests `/api/public/verify/undefined`; the verification API hashes `undefined`, finds no active matching `token_hash`, returns its intentional generic `404`, and the page calls `notFound()`.

Investigation record:

`05_DEBUGGING/investigations/Chat37_Node7_Phase1a_PublicShare404_Report.md`

## Authorized Change

Modify only the public share page route as required to await the Next.js 16 dynamic-route params contract.

Target file:

`src/app/share/[token]/page.tsx`

The route should receive params as:

`Promise<{ token: string }>`

and must await params before using the token:

`const { token } = await params;`

Then pass that resolved token to the existing verification-data function.

## Strict Boundaries

- Do NOT change the public verification API route unless a build/type check proves an independent issue that must be reported separately.
- Do NOT change token generation or hashing.
- Do NOT change `trip_public_shares` schema or database logic.
- Do NOT change authentication, RLS, rate limiting, or security behavior.
- Do NOT redesign the public share page.
- Do NOT modify delivery-event mappings as part of this fix.
- Do NOT perform unrelated cleanup/refactoring.
- Do NOT add a second evidence source.
- Do NOT expose secrets or raw public tokens in the report.

## Implementation Requirements

1. Inspect the target file before editing.
2. Apply the smallest possible change needed for Next.js 16 compatibility.
3. Preserve all existing UI behavior and verification response handling.
4. Run the project's appropriate type/build checks after the change.
5. If practical, add or run a focused verification that demonstrates the resolved token is passed rather than `undefined`.
6. Do not claim production success solely from a local build.

## Required Verification

At minimum verify:

- TypeScript/build passes.
- `src/app/share/[token]/page.tsx` now awaits dynamic `params`.
- The verification call receives the resolved token.
- No unintended files are modified.
- Existing backend verification behavior remains unchanged.

If production testing is available through the normal project workflow, test the public share URL with a newly generated active share. Do not include the raw token in the report.

## Required Antigravity Report

Create the implementation report in:

`03_IMPLEMENTATION/implementation_reports/Chat37_Node7_Phase1a_PublicShare404_LocalizedParamsFix_Implementation_Report.md`

Report:

1. Objective
2. Root cause reference
3. Files changed
4. Exact change summary
5. Build/type/test commands run
6. Results
7. Production verification result, if performed
8. Unintended-change check
9. Remaining issues, if any
10. Final status: `IMPLEMENTED — AWAITING AYUSH MANUAL VERIFICATION` unless production behavior has been verified through the approved workflow.

Do not push or create unrelated commits unless Ayush separately authorizes the push step.
