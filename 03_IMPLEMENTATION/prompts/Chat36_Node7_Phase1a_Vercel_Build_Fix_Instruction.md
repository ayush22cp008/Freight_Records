# Chat36 — Node 7 Phase 1a Vercel Build Fix Instruction

## Objective
Fix the Vercel production build failure blocking deployment of the completed Phase 1a Company Public-Share UI.

## Verified Failure
Vercel TypeScript validation reports incompatible Next.js route-handler `params` types in:
- `src/app/api/public/verify/[token]/route.ts`
- `src/app/api/trips/[tripId]/public-share/route.ts`

The deployed Next.js type contract expects `params` as a Promise, while the current handlers type/use it as a synchronous object.

## Required Fix
1. Inspect the current Next.js version and existing route-handler conventions in the repository.
2. Correct the affected route-handler `params` typing and access pattern to match the installed Next.js contract.
3. Preserve all existing Phase 1a behavior: authorization, token hashing, rate limiting, privacy projection, AI summary generation, create/replace/revoke behavior, and generic public 404 behavior.
4. Do not change the database architecture or introduce unrelated refactoring.

## Verification
- Run the relevant TypeScript checks.
- Run `npm run build` successfully.
- Confirm both affected routes compile without TS2344 errors.
- Confirm no regression to Company PublicShareManager integration.
- Commit and push the fix to `main` so Vercel can deploy it.

## Scope Guardrails
- Phase 1a deployment blocker only.
- Do not start Phase 1b.
- Do not modify Driver Trip/Timeline behavior.
- Do not create Chat37.
- If another build error appears, inspect and report it rather than silently expanding scope.

After a successful build and push, STOP for Ayush to confirm the Vercel production deployment and continue manual Phase 1a verification.
