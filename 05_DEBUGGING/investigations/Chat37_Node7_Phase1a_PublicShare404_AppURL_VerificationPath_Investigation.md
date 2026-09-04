# Chat37 — Node7 Phase1a — Public Share 404 — App URL / Verification Path Investigation

## Status
INVESTIGATION ONLY — NO SOURCE CHANGES AUTHORIZED

## Context
The production deployment for the localized Next.js 16 route-params fix is Ready on the production environment, but opening a valid public share URL still returns the generic Next.js 404 page.

Previous code-side investigation verified that `src/app/share/[token]/page.tsx` synchronously accessed Next.js 16 dynamic route `params`, causing `token` to become `undefined`, the page to call `/api/public/verify/undefined`, and the verification API to return 404. The page was then corrected to await `params` and the fix was deployed as commit `a71a66f40b261fc32aab82c5d080423c5cc7dca3`.

Manual production verification after that deployment still reports the same generic 404. Therefore the previous root cause is confirmed but is not sufficient to explain the remaining production failure.

## Investigation Objective
Trace the complete production execution path from the public share page through its server-side verification fetch and determine exactly where the remaining 404 is produced.

## Required Trace
1. Confirm the deployed production source corresponds to commit `a71a66f40b261fc32aab82c5d080423c5cc7dca3` and contains the awaited `params` fix.
2. Inspect `src/app/share/[token]/page.tsx` and trace `getVerificationData(token)`.
3. Determine whether `NEXT_PUBLIC_APP_URL` is present and correctly configured for Vercel Production, without exposing its value if it contains sensitive infrastructure information.
4. Determine the effective server-side fetch target constructed from `NEXT_PUBLIC_APP_URL` and `/api/public/verify/${token}`. Do not include real tokens or secrets in the report.
5. Verify whether the server-side page fetch reaches the production verification API and what HTTP status/result it receives.
6. If the verification API itself returns 404, continue the trace into `src/app/api/public/verify/[token]/route.ts` and determine whether the failure is caused by token validation, token hashing, active-share lookup, trip lookup, or another API branch.
7. Compare the direct browser-visible verification API behavior with the page's server-side fetch behavior, where feasible, to distinguish routing/environment failure from database/token failure.
8. Identify every code path that converts fetch exceptions or non-OK responses into `null` and then `notFound()`, because this can mask an upstream runtime/configuration failure as a generic page 404.
9. Record whether the observed behavior is caused by runtime configuration, route resolution, API response, database lookup, deployment mismatch, or another verified cause.

## Explicit Investigation Boundary
- Do NOT modify source code.
- Do NOT modify Vercel environment variables.
- Do NOT redeploy.
- Do NOT rotate/regenerate public-share tokens solely for diagnosis.
- Do NOT reopen Nodes 1–6.
- Do NOT begin Phase1b or Phase3 work.
- Do NOT introduce a second evidence source.
- Do NOT expose raw public-share tokens, credentials, environment secrets, database credentials, or private URLs in the report.

## Evidence Requirements
Use confidence tags:
- VERIFIED — directly demonstrated by source/runtime/deployment evidence.
- INFERRED — strongly indicated but not directly demonstrated.
- UNKNOWN — not yet observable or not available from the investigation environment.

The report must distinguish:
- source/deployment facts,
- runtime/configuration observations,
- API response evidence,
- database lookup evidence,
- inference,
- remaining unknowns.

## Expected Outcome
Produce a concise investigation report identifying the exact remaining failure point, or explicitly state the narrowest unresolved boundary if production runtime evidence cannot be obtained. Do not propose or implement a fix until the failure point is established.

## Required Report
`05_DEBUGGING/investigations/Chat37_Node7_Phase1a_PublicShare404_AppURL_VerificationPath_Report.md`
