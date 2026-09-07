# Chat43 — Day 17 — Node 7 — Phase 1b — Driver Intermittent Photo Upload Failure Resolution Report

## 1. Investigation Conclusion

The investigation into the intermittent Driver photo upload failure during event submission has been completed. The issue is a **Payload Size Limitation (413 Payload Too Large) and/or Timeout (504 Gateway Timeout)** inherent to Vercel's Serverless Function architecture when handling high-resolution mobile camera uploads.

## 2. Root Cause Analysis

In `src/app/api/upload-photo/route.ts`, the application handles file uploads by reading the entire file into memory before pushing it to Supabase Storage:
```typescript
    const formData = await request.formData();
    const file = formData.get('photo') as File | null;
    // ...
    const arrayBuffer = await file.arrayBuffer();
    const buffer = Buffer.from(arrayBuffer);
```

**Why this fails intermittently:**
1. **Vercel Payload Limit:** Vercel Serverless Functions have a hard body size limit (typically 4.5 MB). Modern mobile cameras routinely produce JPEG/HEIC images ranging from 3 MB to 10 MB depending on lighting and detail. When a driver uploads a photo exceeding this 4.5 MB limit, Vercel rejects the request with a `413 Payload Too Large` error before the route handler can even execute properly.
2. **Memory/Timeout Issues:** Converting a large file to an `ArrayBuffer` and then to a Node `Buffer` inside a Serverless Function consumes significant memory. On a slow mobile connection, uploading a 5MB+ file to the serverless function and then bridging it to Supabase can exceed the default 10-second timeout on Vercel's hobby tier, resulting in a `504 Gateway Timeout`.

This explains why the issue is **intermittent**: if a driver captures a visually simple scene or the camera auto-compresses the photo to just under 4.5 MB, the upload succeeds. If the photo is slightly larger, it fails. A retry with a new photo might yield a smaller file, leading to the observed "success on retry" behavior.

## 3. Boundary Verification
- **Shared Upload Utility:** The issue is located in the shared `src/app/api/upload-photo/route.ts` boundary. Because all event clients (Arrival, Load, Departure, etc.) use this route, the intermittent failure can occur across the entire application workflow.
- **Security Boundary:** The authentication and trip authorization logic (`eq('driver_id', driver.id)`) is fully functional and is not the cause of the intermittent failure.
- **Supabase Storage:** The `event-photos` bucket is properly configured; the failure occurs during the HTTP transfer to the Next.js API route, not within Supabase itself.

## 4. Recommendation for Fix
To resolve this issue permanently without changing backend APIs or risking serverless limits, we must bypass the Next.js API route for the actual file binary transfer.

**Proposed Approach:**
1. **Client-Side Compression:** Implement a lightweight client-side image compression utility (e.g., using an HTML5 Canvas to resize/compress the photo to a maximum width/height and quality before appending it to `FormData`). This ensures the payload reliably stays under the 4.5 MB limit.
2. **Direct-to-Storage (Optional alternative):** Use Supabase's client-side direct upload via `supabase.storage.from('event-photos').upload(...)` combined with Row Level Security (RLS) policies. However, since the prompt forbids RLS changes, **Client-Side Compression** is the only strictly compliant and authorized fix.

**Authorized Fix Scope:**
Implement client-side compression inside `src/lib/capture/uploadPhoto.ts` before the file is appended to `FormData`. This isolates the fix entirely to the frontend utility and respects all protected boundaries.

## 5. Decision Status
**STATUS: INVESTIGATION COMPLETE — READY FOR IMPLEMENTATION**
The investigation confirms the issue is a payload/timeout limitation in the serverless upload route caused by uncompressed mobile photos. A focused implementation prompt can now authorize adding client-side image compression to the shared upload helper.
