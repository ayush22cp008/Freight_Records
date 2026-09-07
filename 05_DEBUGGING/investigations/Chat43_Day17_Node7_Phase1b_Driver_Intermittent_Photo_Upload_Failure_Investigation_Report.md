# Chat43 — Day 17 — Node 7 — Phase 1b — Driver Intermittent Photo Upload Failure Investigation Report

## 1. Investigation Status

**STATUS: INVESTIGATION OPEN — NO FIX AUTHORIZED YET**

**Portal:** Driver

**Phase:** Node 7 — Phase 1b — Driver UI/UX / regression verification

**Scope:** Intermittent failure and long delay during Driver photo upload when submitting an event, currently observed at the `Arrival at Delivery` flow.

**Important:** This investigation is separate from the completed Timeline historical-trip selection issue and the resolved mobile photo-layout issue.

---

## 2. New Manual Observation

Ayush reported an intermittent photo-upload problem while using the deployed Driver application on mobile.

Observed behavior:

```text
Select/capture photo
        ↓
Submit event
        ↓
Long waiting period
        ↓
Sometimes: event succeeds
Sometimes: "Failed to upload photo"
```

The provided screenshot shows the `Record Arrival at Delivery: navsari` page with:

- a selected image filename;
- an error box reading **"Failed to upload photo"**;
- the submit button still available;
- no successful arrival confirmation for that attempt.

Ayush also reports that after refreshing/retrying, the same flow can succeed, while at other times it fails again.

This makes the defect **intermittent**, rather than a deterministic "photo upload always fails" condition.

---

## 3. Separate Observation — Page/Workflow State

The screenshot showing `Record Arrival at Delivery: navsari` is not by itself evidence that the application unexpectedly changed the trip.

The project records state that the Driver dashboard transitions to `Record Arrival at Delivery` after the `ARRIVED_AT_DELIVERY` milestone is reached/handled in the delivery lifecycle. The milestone implementation uses the existing GPS, server-time, and `uploadPhoto` infrastructure. fileciteturn329file0

Therefore:

- **The page being "Record Arrival at Delivery" is not classified as a defect from the screenshot alone.**
- **The intermittent "Failed to upload photo" behavior is the actual defect under investigation.**

---

## 4. Existing Upload Architecture

The current application uses a shared upload utility:

```text
Event client
    ↓
uploadPhoto(file, tripId)
    ↓
POST /api/upload-photo
    ↓
Authenticated user verification
    ↓
Driver/company + trip authorization
    ↓
Supabase Storage: event-photos
    ↓
Public URL returned
    ↓
Event API submission
```

The current `uploadPhoto.ts` helper creates `FormData`, sends the photo and `trip_id` to `/api/upload-photo`, and throws the server-provided error or the generic `Failed to upload photo` message when the upload response is not successful. fileciteturn297file0

The current upload route authenticates the user, verifies the relevant Driver/Company identity and trip ownership, reads the uploaded file into an `ArrayBuffer`/`Buffer`, uploads it to the `event-photos` bucket, and returns a public URL. On a Supabase storage error it deliberately returns the generic HTTP 500 response **`Failed to upload photo`**. fileciteturn299file0

The secured-storage implementation records that `event-photos` is the established shared evidence bucket and that Driver upload authorization is bound to the authenticated Driver and trip. fileciteturn294file0

---

## 5. Why the Current Error Message Is Not Enough

The browser-visible message:

```text
Failed to upload photo
```

does not identify the underlying failure.

The current upload route logs the actual Supabase storage error server-side but intentionally returns only the generic message to the client. fileciteturn299file0

Therefore the present evidence does **not** establish whether the intermittent failure is caused by:

1. transient mobile/network connectivity;
2. request/serverless execution delay or timeout;
3. Supabase Storage upload failure;
4. file size or file characteristics;
5. storage service response failure;
6. authentication/session timing;
7. trip authorization/state changing during a long upload;
8. deployment/runtime environment behavior;
9. another failure in the shared upload path.

No one of these should be declared the root cause without evidence.

---

## 6. Important Source-Level Clues

### Clue A — Shared utility

The same `uploadPhoto(file, tripId)` utility is used by multiple event clients, including Arrival, Check-in, Departure, Load, In-Transit, Goods Unloaded, Pickup Departed, Delivery Departed, and destination/receiver-related flows. fileciteturn296file1 fileciteturn296file2 fileciteturn296file3 fileciteturn296file4 fileciteturn296file6 fileciteturn296file7 fileciteturn296file8 fileciteturn296file9 fileciteturn296file10

This means a shared upload failure could affect more than Arrival at Delivery.

### Clue B — Arrival at Delivery reuses the shared utility

`ArrivedAtDeliveryClient.tsx` calls `uploadPhoto(photoFile, tripId)` before it submits the `ARRIVED_AT_DELIVERY` event. fileciteturn303file0

### Clue C — Current route has no explicit client-side upload timeout

The visible helper awaits the `fetch('/api/upload-photo')` response directly. The current source shown in the repository contains no explicit upload timeout/retry/diagnostic classification in the helper. fileciteturn297file0

This is a diagnostic observation only, **not proof that a timeout is the root cause**.

### Clue D — Current route converts the entire file into memory before storage upload

The API route performs `await file.arrayBuffer()` followed by `Buffer.from(arrayBuffer)` before calling Supabase Storage. fileciteturn299file0

Again, this is a potential investigation target, not a confirmed defect.

---

## 7. Historical Baseline

The project previously had a deterministic photo-upload failure caused by the missing `event-photos` bucket. That historical problem was later addressed through the secure-storage work: the bucket was created and the upload route was secured, and the project checkpoint subsequently recorded check-in photo upload and other evidence flows as operationally verified. fileciteturn291file0 fileciteturn294file0 fileciteturn314file0

Therefore, the current intermittent behavior must **not** automatically be classified as the old missing-bucket defect.

The fact that the same photo can succeed after a retry is especially important evidence against treating this as a simple deterministic missing-resource failure.

---

## 8. Root-Cause Status

**CURRENT ROOT CAUSE: UNKNOWN.**

The screenshots prove the symptom, but not the underlying failure mechanism.

The investigation must obtain the actual failing request/response or server-side error before authorizing a code change.

---

## 9. Required Source/Runtime Investigation

### A. Reproduce with controlled tests

Test the same event flow repeatedly and record:

- photo file size;
- file type/extension;
- approximate upload duration;
- whether the failure occurs before or after GPS/server-time acquisition;
- HTTP status for `/api/upload-photo`;
- response body;
- whether retrying the exact same file succeeds;
- whether a newly captured smaller photo behaves differently.

### B. Inspect browser Network evidence

For a failed attempt, capture the `/api/upload-photo` request and determine:

- status code;
- request duration;
- request payload/body size if visible;
- response body;
- whether the request is aborted/cancelled;
- whether the failure is a network error rather than an HTTP response.

For a successful attempt, capture the same information and compare.

### C. Inspect server/runtime logs

Correlate the failed request with:

```text
Supabase upload error
Upload route error
```

The current route logs the underlying error server-side before returning the generic 500 message. fileciteturn299file0

The actual logged error should determine whether the failure originates from Storage, request processing, authorization, or runtime/network behavior.

### D. Compare event clients

Because multiple event clients reuse `uploadPhoto`, determine whether the issue is:

- Arrival-at-Delivery-specific, or
- shared across the upload infrastructure.

### E. Compare environments

Test both:

- deployed Vercel application;
- local development application;

and establish whether the intermittent behavior occurs in both.

---

## 10. Diagnostic Test Matrix

| Test | Required evidence | Status |
|---|---|---|
| Arrival at Delivery + small photo | success/failure + duration + HTTP status | REQUIRED |
| Arrival at Delivery + larger photo | success/failure + duration + HTTP status | REQUIRED |
| Retry same photo after failure | whether exact same file succeeds | REQUIRED |
| Arrival at Delivery without photo | verify event path independent of upload | REQUIRED |
| Check-in + photo | compare shared upload path | REQUIRED |
| Departure + photo | compare shared upload path | REQUIRED |
| Goods Unloaded + photo | compare shared upload path | REQUIRED |
| Localhost upload | compare runtime | REQUIRED |
| Vercel upload | compare runtime | REQUIRED |
| Failed request server log | exact underlying error | REQUIRED |

---

## 11. Protected Boundaries

Until the root cause is established, do **NOT** change:

- database schema;
- Storage bucket configuration;
- Storage policies/RLS;
- authentication/authorization;
- event APIs;
- event vocabulary;
- trip lifecycle semantics;
- evidence model;
- photo storage architecture;
- Timeline behavior;
- Completed Trips behavior;
- Company portal;
- Reviewer portal.

A frontend-only change should not be assumed merely because the visible error appears in a client component. The actual failure occurs in the shared upload path and may involve the server/storage boundary.

---

## 12. Implementation Authorization

**NO IMPLEMENTATION IS AUTHORIZED YET.**

Do not add retries, compression, file-size limits, upload timeouts, new storage logic, or error-message changes until the failing request has been classified.

In particular:

- Do not blindly add retry logic to hide a server/storage failure.
- Do not compress images without evidence that file size is the cause.
- Do not change Storage/RLS because the old missing-bucket issue existed historically.
- Do not change event APIs because the failure occurs before event submission when photo upload fails.

---

## 13. Acceptance Criteria for Closing the Investigation

The investigation can be marked complete only when:

1. A failed upload is reproducible or its runtime evidence is captured.
2. The exact HTTP/network failure is identified.
3. The underlying server/storage error is identified where applicable.
4. The cause is classified as client/network, runtime/server, storage, authorization, file characteristics, or another verified category.
5. The smallest safe fix boundary is defined.
6. Existing security/authorization behavior is preserved.
7. Existing successful photo-upload flows remain protected from regression.

Only after these criteria are satisfied should a separate implementation prompt be created.

---

## 14. Current Decision

**DECISION: INVESTIGATE FIRST — DO NOT FIX YET.**

The new intermittent failure is materially different from the historical deterministic missing-bucket issue and must be diagnosed from actual request/runtime evidence.

The correct sequence is:

**Observe → Reproduce → Capture request/runtime evidence → Determine root cause → Decide scope → Implement → Build/Test → Ayush manual verification → Record resolution → Close Driver.**

No unrelated Driver functionality should be modified while this investigation is open.
