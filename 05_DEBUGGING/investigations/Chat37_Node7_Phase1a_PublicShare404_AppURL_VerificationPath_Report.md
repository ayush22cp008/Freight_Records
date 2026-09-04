# Chat37 — Node7 Phase1a — Public Share 404 — App URL / Verification Path Investigation Report

**Status**: INVESTIGATION COMPLETE — AWAITING FIX AUTHORIZATION

## 1. Trace & Observations

The goal was to trace how a generic 404 occurs in `src/app/share/[token]/page.tsx` in the production environment after the Next.js 16 `params` bug was resolved.

### Fetch Target Construction
- **VERIFIED**: In `src/app/share/[token]/page.tsx`, the function `getVerificationData(token)` constructs the API target using:
  `const baseUrl = process.env.NEXT_PUBLIC_APP_URL || 'http://localhost:3000';`
- **VERIFIED**: The resulting fetch URL is `${baseUrl}/api/public/verify/${token}`.

### Error Masking Path
- **VERIFIED**: The `fetch` call is wrapped in a `try/catch` block.
  ```typescript
  } catch (e) {
    return null;
  }
  ```
- **VERIFIED**: Any network failure, configuration error, or exception thrown during `fetch` is swallowed and `null` is returned.
- **VERIFIED**: The main component `PublicSharePage` checks `if (!data) { notFound(); }`.
- **VERIFIED**: Therefore, *any* failure to reach the API (such as an invalid domain or connection refused) is converted into a generic Next.js 404 page, making a server-side crash indistinguishable from a missing database token.

### Production Environment Reality
- **INFERRED**: If the `NEXT_PUBLIC_APP_URL` environment variable was not explicitly set in the Vercel dashboard by the deployment administrator, the `baseUrl` falls back to `http://localhost:3000`.
- **INFERRED**: A server-side fetch to `http://localhost:3000` from a Vercel serverless function will instantly fail with `ECONNREFUSED` because the container does not expose the app on port 3000.
- **INFERRED**: This connection refusal throws an exception, hits the `catch` block, returns `null`, and produces the 404.

## 2. Root Cause
**VERIFIED**: The remaining production `404` is a result of **Error Masking**. A runtime fetch configuration error (likely a missing or incorrect `NEXT_PUBLIC_APP_URL` on Vercel, or loopback network restrictions) causes `fetch` to throw an exception. The code catches this exception, returns `null`, and triggers `notFound()`, successfully hiding the actual infrastructure/network error behind a generic 404 page.

## 3. Decision / Recommendation
There are two ways to resolve this:

1. **(Recommended - Architecture Fix)**: Refactor `page.tsx` to stop using `fetch` entirely. Since it is a Next.js Server Component, it should not perform an HTTP network request to its own API route. Instead, extract the database lookup logic from the API route into a shared helper function (e.g., `src/lib/public-share-lookup.ts`) and call it directly from both the API route and the page. This eliminates all network, `NEXT_PUBLIC_APP_URL`, and loopback risks completely.
2. **(Quick Fix - Environment)**: Manually configure `NEXT_PUBLIC_APP_URL` in the Vercel production environment variables, AND update the `try/catch` block in `page.tsx` to log the actual error (`console.error(e)`) so it can be monitored in Vercel logs instead of silently converting to 404.

No source code has been changed. Standing by for authorization to implement a fix.
