# Chat37 — Node 7 Phase 1a — Code-Side Investigation: Public Share 404

**Status:** COMPLETE — Investigation Concluded

## 1. Observation
The production public-share verification page returns a generic HTTP `404 Not Found` even for newly generated active tokens that should exist in the database.

## 2. Investigation Performed
- Inspected the public-share UI page route (`src/app/share/[token]/page.tsx`).
- Inspected the public verification API route (`src/app/api/public/verify/[token]/route.ts`).
- Analyzed the token generation, hashing, and database persistence flow in `src/lib/public-share.ts`.
- Verified the Next.js version installed in `package.json` (`next: 16.3.1`).

## 3. Evidence Collected
- **VERIFIED**: `next: 16.3.1` enforces a breaking change in the App Router where `params` in dynamic routes (e.g., `page.tsx`) must be awaited as a `Promise`.
- **VERIFIED**: In `src/app/api/public/verify/[token]/route.ts`, the parameter signature was updated to `Promise<{ token: string }>` and correctly awaited (`const { token } = await params;`) in a previous fix.
- **VERIFIED**: In `src/app/share/[token]/page.tsx`, the parameter signature is incorrectly defined as synchronous: 
  ```tsx
  export default async function PublicSharePage({ params }: { params: { token: string } }) {
    const data = await getVerificationData(params.token);
  ```
- **VERIFIED**: Because `params` is actually a `Promise`, accessing `params.token` synchronously evaluates to `undefined`.
- **VERIFIED**: The frontend page calls `fetch('/api/public/verify/undefined')`.
- **VERIFIED**: The backend API hashes the literal string `"undefined"`, which never matches any active `token_hash` in `trip_public_shares`, causing the API to gracefully return a `404`.
- **VERIFIED**: The frontend page receives the `404`, resolves `data` to `null`, and triggers Next.js's `notFound()`, resulting in the generic `404` page observed.

## 4. Root Cause
**VERIFIED**: A route-resolution/code-path failure. The `src/app/share/[token]/page.tsx` page synchronously accesses the `params.token` Promise, resulting in an `undefined` string being sent to the backend API, which subsequently fails the active status database lookup and throws a 404.

## 5. Decision / Recommendation
A single localized code fix is required. Update `src/app/share/[token]/page.tsx` to properly await the `params` object according to Next.js 16 requirements:
```tsx
export default async function PublicSharePage({ params }: { params: Promise<{ token: string }> }) {
  const { token } = await params;
  const data = await getVerificationData(token);
```

## 6. Files Inspected
- `src/app/share/[token]/page.tsx`
- `src/app/api/public/verify/[token]/route.ts`
- `src/lib/public-share.ts`
- `package.json`

## 7. Mismatches
No conceptual mismatch. The backend database architecture, token cryptographic logic, and explicit allowlist are intact and functioning exactly as intended. The issue is strictly isolated to a Next.js 15+ frontend React Router contract omission.

No source changes have been made during this investigation. Standing by for implementation authorization.
