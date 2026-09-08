# Chat43 — Day 17 — Node 7 — Phase 1b — Driver Photo Upload Persistent Retry Failure Resolution Report

## 1. Investigation Conclusion

The investigation into the persistent retry failure has been successfully completed. 

The reported behavior—where an initial upload fails, and subsequent retries of the same photo continue to fail until a page refresh—is entirely explained by the interaction between the React client state and Vercel's serverless infrastructure limits. 

The root cause is a **deterministic rejection of the identical payload**, not a broken client state handler or a stale connection.

## 2. Core Questions Answered

### Question A: Why does the initial photo upload fail?
As established in the related intermittent failure investigation, the application runs on Vercel, which imposes strict Serverless Function body size limits (typically 4.5MB) and timeouts (10 seconds on Hobby). Modern mobile cameras routinely produce high-resolution photos (3MB-8MB). When a driver captures a photo that exceeds these limits, Vercel infrastructure rejects the request with a `413 Payload Too Large` or `504 Gateway Timeout` before the Next.js API route (`/api/upload-photo`) can successfully parse and proxy the image to Supabase.

### Question B: Why do subsequent retry attempts continue failing after the first failure?
This is **Category 4 (Server/runtime defect repeating on identical input).**
When the first upload fails, the React `handleSubmit` logic cleanly catches the error, sets the error message, and re-enables the submit button. The client state is *not* broken or deadlocked.

When the user clicks "Submit" to retry, the handler executes again and calls `fetch()` to initiate a genuinely new network request. However, it passes the **exact same `File` object** stored in the React `photoFile` state. Because the file size and network conditions are unchanged, this identical payload hits the exact same Vercel 413/504 limit and is deterministically rejected again. The retry fails persistently because the input is persistently too large.

### Why does a refresh "fix" it?
A full page refresh clears the React component state, including the stored `photoFile`. The driver is forced to interact with the file input again (`capture="environment"`). By taking a *new* photo, the mobile OS may generate an image with slightly different lighting, complexity, or auto-compression, resulting in a file size that happens to fall under the 4.5MB limit. Thus, the upload suddenly succeeds, giving the illusion that the "refresh" fixed a broken client state.

## 3. Evidence Matrix & State Trace
- **Client Handler Execution:** Verified. `GoodsUnloadedClient` (and sibling events) properly clear `error` and reset `loading=true` at the start of `handleSubmit`.
- **File Object Retention:** Verified. The `File` object in state is perfectly valid across retries, but its size (e.g., 6.2MB) remains constant.
- **Request Generation:** Verified. A new `FormData` and `fetch` call is generated inside `uploadPhoto.ts` on every retry.
- **Underlying Error:** A consistent `413` or `504` from Vercel infrastructure.

## 4. Fix Recommendation

**"Adding a retry loop" is NOT an acceptable fix**, because a retry with the exact same oversized file will just fail again.

The **only correct and fully authorized fix** is to address the payload size itself. We must implement **client-side image compression** before the file is uploaded.

**Proposed Implementation Scope:**
Modify the shared utility `src/lib/capture/uploadPhoto.ts` to compress the `File` using the browser's native HTML5 Canvas API (resizing it to a reasonable maximum width/height and JPEG quality, e.g., max 1920px width and 0.8 quality). This ensures that *every* photo, regardless of the camera's megapixel count, is reduced to well under 1MB before `FormData` is created. 

This single, isolated fix will eliminate the initial 413/504 failures and, consequently, eradicate the persistent retry failure loop.

## 5. Decision Status
**STATUS: INVESTIGATION COMPLETE — READY FOR IMPLEMENTATION**
The investigation confirms the persistent failure is caused by retrying an identical oversized payload against strict serverless limits. A focused implementation prompt can now authorize adding client-side image compression to the shared `uploadPhoto.ts` utility.
