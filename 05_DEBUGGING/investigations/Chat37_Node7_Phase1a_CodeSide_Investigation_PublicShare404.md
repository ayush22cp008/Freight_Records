# Chat37 — Node 7 Phase 1a — Code-Side Investigation: Public Share 404

**Chat:** Chat 37  
**Hackathon Day:** Day 14  
**Node:** Node 7 — AI + Final Integration + Demo  
**Phase:** Phase 1a — Public Shareable Evidence  
**Investigation type:** Code-side / runtime-path diagnosis  
**Status:** OPEN — investigation only

## Objective

Determine the real cause of the production public-share verification `404` from the application code and runtime route path before proposing or applying any fix.

Do not treat the HTTP 404 itself as proof of the underlying cause. The implementation intentionally uses a generic 404 for invalid, revoked, or unusable public tokens, so the investigation must identify which internal condition produces the observed result.

## Current known context

- Public Evidence Share UI is implemented and visible.
- Token generation and SHA-256 token hashing are recorded as implemented.
- `trip_public_shares` persistence is recorded as implemented.
- Production public verification currently returns 404.
- The public `/share/[token]` page also requires code-level reconciliation.
- Existing records identify possible code-side concerns around the Next.js dynamic-route `params` contract, delivery-event mapping, and public verification lookup behavior.
- Do not perform blind redeployments.
- Do not modify source code during this investigation.
- Do not reopen Nodes 1–6.

## Investigation scope

Antigravity must inspect the current source repository and determine, with evidence:

1. Exact public-share page route and how it obtains/uses `token`.
2. Exact public verification API route and how its dynamic `params` are resolved.
3. Installed Next.js version and the route-handler contract actually required by that version.
4. Every code path in the verification API that can return HTTP 404.
5. Token generation path versus verification path:
   - token encoding
   - hashing algorithm
   - hash input
   - stored representation
   - lookup field
6. Exact `trip_public_shares` query performed by the verification API, including filters such as active status.
7. Whether the API route can be reached and executed in the deployed application, independently of whether the database lookup succeeds.
8. Whether any route/path/parameter mismatch causes the request to fail before the intended verification logic executes.
9. Whether delivery-event name mapping can affect the request's 404 outcome, or is only a later response/data issue.
10. Whether any error handling collapses an underlying database/runtime exception into the generic 404.
11. Whether the current source differs from the implementation that was previously reported as deployed.
12. Identify the smallest set of direct evidence needed to distinguish:
    - route-resolution/code-path failure
    - token/parameter handling failure
    - database lookup failure
    - active-status/filter failure
    - downstream evidence-query failure
    - generic error masking
    - other verified cause

## Evidence requirements

Collect concrete evidence from the source repository/runtime inspection, such as:

- exact file paths
- relevant function/route definitions
- package/version information
- grep/search results
- build/runtime output where useful
- database query shape as represented in source
- route resolution/build output where available

Do not expose secrets, raw credentials, or the raw public token in the report.

## Required investigation pipeline

```text
OBSERVATION
→ INVESTIGATION
→ EVIDENCE
→ ROOT CAUSE
→ DECISION
```

This record ends at the diagnosis/decision stage. A separate implementation instruction must be created only after the root cause is established and the decision is approved.

## Confidence tagging

For each material finding, mark it:

- **VERIFIED** — directly confirmed by source/runtime evidence.
- **INFERRED** — reasonable conclusion not yet directly confirmed.
- **UNKNOWN** — evidence is insufficient.

Do not label a suspected cause as VERIFIED.

## Required Antigravity report

Return a concise implementation report to the Records repository containing:

1. Observation
2. Investigation performed
3. Evidence collected
4. Root cause — or explicitly `UNKNOWN` if not proven
5. Decision/recommendation
6. Files inspected
7. Any mismatch between source and expected/deployed behavior
8. No source changes unless separately authorized by a later implementation instruction

The active reasoning brain will review that report before any fix instruction is created.
