# Freight Hackathon - Antigravity Handoff: Chat9 Master Prompt

## 1. Project Context & Status
**Current Phase:** Node 7 — Phase 1b (Stage 1 Complete, moving to Stage 2)

We have successfully completed a comprehensive round of bug fixing and stabilization for the **Driver Portal** (Phase 1b Stage 1). The application is deployed on Vercel and all recent codebase changes have been committed and synced to both repositories (`freight_hackathon` for the Next.js app, `Freight_Records` for documentation).

## 2. Work Completed in Previous Session

During the previous session, we successfully resolved several critical regressions and architectural constraints in the Driver Portal:

### A. Next.js 15+ `searchParams` Promise Regression
- **Issue:** The Completed Trips -> View Timeline flow would always fall back to displaying the driver's most recent active trip instead of the selected historical trip.
- **Fix:** Next.js 16.3.1 treats `searchParams` as a Promise. We updated `src/app/(authenticated)/timeline/page.tsx` to properly `await` the `searchParams` to extract the `tripId`, restoring correct historical navigation.

### B. Mobile Photo Layout Horizontal Overflow
- **Issue:** Event success screens with uploaded photo evidence were rendering a large black space on the right side of mobile viewports because the photo `<img>` used `max-w-sm` without `w-full`.
- **Fix:** We corrected this across all 10 affected screens: Timeline, Arrival, Goods Unloaded, Pickup Departed, Load, In-Transit, Departure, Delivery Departed, Check-in, and Arrived at Delivery. We added `w-full max-w-sm` to perfectly contain the image on mobile while preventing excessive stretching on desktop.

### C. Persistent Photo Upload Failure
- **Issue:** Driver photo uploads intermittently failed with "Failed to upload photo." Crucially, after a first failure, subsequent retries of the same photo *always* failed until the page was refreshed.
- **Root Cause:** Modern mobile cameras produced payloads exceeding Vercel's Serverless Function limit (4.5MB). The persistent retry failure occurred because React state reused the exact same oversized `File` payload, continuously hitting the exact same serverless 413/504 limit on every retry.
- **Fix:** Implemented lightweight client-side HTML5 Canvas image compression directly inside `src/lib/capture/uploadPhoto.ts`. All photos are now seamlessly resized to a maximum of 1920px (at 0.8 JPEG quality) *before* hitting the Vercel infrastructure, completely eliminating the upload payload limitation and the associated retry loop.

## 3. Strict Development Rules

1. **Rule of Documentation:** Every time you build a report, investigation, or implementation plan, you MUST save it as a separate markdown file in the appropriate directory (`05_DEBUGGING/investigations/` or `03_IMPLEMENTATION/implementation_reports/` inside `Freight_Records`). You must then stage, commit, push the file to GitHub, and immediately `rm` (delete) it from the local filesystem. Always provide the raw GitHub URL in a copy-paste block in your response.
2. **Rule of Direct Action:** If the user asks for an "investigation report" or an "implementation report", you MUST skip the "implementation plan" phase. Just do the investigation, write the report, push it, and delete it locally. Do not block on asking for permission to plan.
3. **Rule of Scope:** Only modify the exact frontend files necessary to solve a bug. Do NOT modify APIs, Database schemas, RLS policies, Authentication logic, or Storage buckets without an explicit architectural decision.

## 4. Current Objectives for New Agent

We are now ready to pivot to **Phase 1b Stage 2**, which focuses on the **Company Portal**. 

Your next tasks will be directed by the user, but expect to be implementing, debugging, or refining the Company Portal interfaces (e.g., Company Dashboard, Company Completion, or Receiver Check-in flows). 

If the user reports any further bugs from the Driver Portal, tackle them utilizing the exact same systematic investigation -> implementation report pattern used in the previous session.

*You have full context. Await the user's next command.*
