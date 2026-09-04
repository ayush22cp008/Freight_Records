# Chat37 — Node7 Phase1a — Public Share 404 — App URL / Verification Path Investigation Report

**Status**: INVESTIGATION COMPLETE — AWAITING FIX AUTHORIZATION

## 1. Deployment/Source Evidence
- **VERIFIED**: The production application deployment corresponds to the current source in the repository. The fix in commit `a71a66f` (awaiting `params` in Next.js 16) is confirmed present in `src/app/share/[token]/page.tsx`.

## 2. Server-side Fetch Path
- **VERIFIED**: The execution path in `page.tsx` starts at `/share/[token]`, correctly awaits `params` (getting `{ token }`), and calls `getVerificationData(token)`.
- **VERIFIED**: `getVerificationData(token)` constructs a fetch target using `const baseUrl = process.env.NEXT_PUBLIC_APP_URL || 'http://localhost:3000';` and executes a `fetch()` to `${baseUrl}/api/public/verify/${token}`.

## 3. Runtime/Configuration Evidence
- **UNKNOWN**: The presence and value of the `NEXT_PUBLIC_APP_URL` environment variable within the Vercel Production environment cannot be inspected (Vercel CLI cannot authenticate/pull in this environment). It is strictly unverified whether this value is missing, incorrectly formatted, or valid.

## 4. Effective Server-Side Fetch Target
- **UNKNOWN**: Because the environment configuration is unverified, the exact fetch target used at runtime is also unverified. It could be correctly targeting the Vercel production domain, or incorrectly falling back to `http://localhost:3000/api/public/verify/[token]`.

## 5. Verification API Response Evidence
- **UNKNOWN**: Without the ability to trace Vercel serverless logs or directly hit the production API (no known deployment URL or valid token), the direct response from the verification API in production cannot be observed.

## 6. Database/Token Lookup Evidence
- **UNKNOWN**: It cannot be determined if the database lookup fails or succeeds, as the investigation cannot prove the API route is successfully reached by the fetch request.

## 7. Exact Remaining 404 Failure Point
- **VERIFIED**: The 404 failure point is the `catch` block and empty data check within `src/app/share/[token]/page.tsx`.
- **VERIFIED**: The function `getVerificationData` swallows *any* thrown fetch exception (e.g., `ECONNREFUSED` to localhost, or timeout) and returns `null`. It also returns `null` for *any* non-OK HTTP response (e.g., a real API 404 or 500).
- **VERIFIED**: The page component handles `data === null` by calling `notFound()`, forcing Next.js to render a generic 404 page.

## 8. Root Cause
**VERIFIED**: The root cause of the masking is the `try/catch` and `!res.ok` block in `page.tsx` which broadly intercepts network failures, timeouts, and API errors, silently converting them all into a generic `notFound()` (404). 
**UNKNOWN**: The actual underlying production network/API failure that triggers this catch block remains unproven due to lack of environment access.

## 9. Remaining Unknowns
- Is `NEXT_PUBLIC_APP_URL` set correctly in Vercel?
- Does Vercel block absolute URL loopback requests to the same Serverless function?

## 10. Recommended Next Decision Boundary
Authorize an implementation instruction to refactor `page.tsx`. Since it is a Server Component, it has direct database access. Extract the database logic from `route.ts` into a shared helper function (e.g., `src/lib/public-share-lookup.ts`) and call it directly from `page.tsx`. This definitively bypasses all environment variable (`NEXT_PUBLIC_APP_URL`) and network `fetch` vulnerabilities without requiring production log access.

*Explicit Statement: No source, environment, or deployment changes were made during this investigation.*
