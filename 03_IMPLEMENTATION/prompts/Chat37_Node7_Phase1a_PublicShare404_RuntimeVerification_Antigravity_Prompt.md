# Chat37 — Node7 Phase1a — Public Share 404 — Runtime Verification Investigation Prompt

## Role
You are Antigravity acting as the implementation/execution agent for the Freight project. This task is **investigation only**. Do not modify source code, environment configuration, or deployment state.

## Current Context
The production deployment containing the Next.js 16 dynamic-route `params` fix is Ready and production still returns the generic 404 for the public share page.

Previous investigation established that `src/app/share/[token]/page.tsx` previously accessed dynamic route `params` synchronously. That bug was fixed and deployed in commit:

`a71a66f40b261fc32aab82c5d080423c5cc7dca3`

The current page now awaits `params`, but the production 404 remains.

The latest investigation identified a strong but **unproven** hypothesis: the page performs a server-side fetch using `NEXT_PUBLIC_APP_URL`, and fetch failures/non-OK responses are swallowed and converted into `notFound()`. This means a runtime/configuration failure can appear as a generic 404.

## Investigation Source of Truth
Read and follow:

`05_DEBUGGING/investigations/Chat37_Node7_Phase1a_PublicShare404_AppURL_VerificationPath_Investigation.md`

Also inspect the current source files involved in the path:

- `src/app/share/[token]/page.tsx`
- `src/app/api/public/verify/[token]/route.ts`
- `src/lib/public-share.ts`

## Objective
Determine the **actual remaining production failure point**. Do not stop at the `NEXT_PUBLIC_APP_URL` hypothesis unless runtime evidence proves it.

## Required Investigation Sequence

### 1. Deployment/source verification
- Confirm production is running the intended deployment/commit containing the `params: Promise<{ token: string }>` fix.
- Confirm the current source on `main` matches that deployed change.
- Record commit/deployment evidence without exposing secrets.

### 2. Server-side fetch path
Trace the exact execution path in `page.tsx`:

`/share/[token]`
→ awaited `params`
→ `getVerificationData(token)`
→ `NEXT_PUBLIC_APP_URL`
→ `/api/public/verify/[token]`

Confirm the code behavior from source.

### 3. Production runtime configuration
Determine whether `NEXT_PUBLIC_APP_URL` is configured for the Vercel **Production** environment.

Important:
- You may inspect configuration presence/state if the available environment/tooling permits it.
- Do NOT expose the actual value in the report if it contains infrastructure details or secrets.
- If the value cannot be inspected, mark it `UNKNOWN`; do not infer it is missing.

### 4. Effective fetch target
Determine what host the server-side page fetch would actually use in production.

Distinguish explicitly between:
- configured production app URL;
- fallback `http://localhost:3000`;
- malformed/incorrect URL;
- unreachable target;
- correct production target.

Do not expose raw public-share tokens.

### 5. Verification API reachability/result
Determine whether the server-side fetch reaches:

`/api/public/verify/[token]`

and, where runtime evidence is available, determine its actual response status/category:

- `200`
- `404`
- `429`
- `500`
- network/fetch exception
- other

If possible, compare the page's server-side fetch path with a direct production request to the verification API using a valid test share token. Do not expose the token in the report.

### 6. If API returns 404
Continue into `src/app/api/public/verify/[token]/route.ts` and determine which branch produces the 404:

- invalid/malformed token;
- token hash mismatch;
- no `ACTIVE` `trip_public_shares` record;
- trip lookup failure;
- another explicit 404 branch.

Use actual evidence wherever possible.

### 7. If API returns 200
If the verification API successfully returns `200`, investigate why the page still reaches `notFound()`.

Trace:
- response parsing;
- JSON handling;
- runtime exceptions after the fetch;
- any other page-level condition causing `notFound()`.

Do not modify the code.

### 8. Error masking
Document every relevant path where:

`fetch exception/non-OK response → null → notFound() → generic 404`

This is important, but distinguish the **verified masking mechanism** from the **unverified cause** that triggers it in production.

## Strict Boundaries

DO NOT:
- modify source files;
- modify `NEXT_PUBLIC_APP_URL` or any other Vercel environment variable;
- redeploy;
- rotate tokens;
- change database records;
- change database schema;
- add logging code;
- refactor the public-share implementation;
- implement a fix;
- reopen Nodes 1–6;
- begin Phase1b or Phase3.

This task ends with evidence and diagnosis only.

## Evidence Discipline
Every important conclusion must use one of these confidence tags:

- **VERIFIED** — directly demonstrated by source, deployment, runtime, or observable API/database evidence.
- **INFERRED** — strongly indicated but not directly demonstrated.
- **UNKNOWN** — evidence unavailable.

Do not label a hypothesis as VERIFIED merely because it is technically plausible.

## Required Report
Update/create exactly this investigation report:

`05_DEBUGGING/investigations/Chat37_Node7_Phase1a_PublicShare404_AppURL_VerificationPath_Report.md`

The report must contain:

1. Investigation status
2. Deployment/source evidence
3. Runtime/configuration evidence
4. Effective server-side fetch target category
5. Verification API response evidence
6. Database/token lookup evidence if reached
7. Exact remaining 404 failure point
8. Root cause with confidence tag
9. What is still unknown, if anything
10. Recommended next decision boundary
11. Explicit statement that no source/environment/deployment changes were made

Do not include raw tokens, credentials, secrets, or sensitive environment values.

## Completion Condition
The task is complete only when the report clearly distinguishes **what is proven** from **what is hypothesized** and identifies the narrowest remaining failure boundary.

Do not implement anything after finding the root cause. Stop after updating the report and return the report path plus a concise execution summary.
