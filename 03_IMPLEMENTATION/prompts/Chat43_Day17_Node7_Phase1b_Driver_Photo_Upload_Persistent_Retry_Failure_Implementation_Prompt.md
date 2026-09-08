# Chat43 — Day 17 — Node 7 — Phase 1b — Driver Photo Upload Persistent Retry Failure Implementation Prompt

## 1. Implementation Authorization

**AUTHORIZED: YES — narrowly scoped Driver photo-upload reliability implementation**

**Portal:** Driver

**Phase:** Node 7 — Phase 1b

**Issue:** Photo upload can fail with `Failed to upload photo`; after the first failed attempt, subsequent retries can continue failing until the page is refreshed. A successful first upload may work normally, while a failed first upload can leave the user effectively unable to recover without refresh.

**Source decision:**
`05_DEBUGGING/investigations/Chat43_Day17_Node7_Phase1b_Driver_Intermittent_Photo_Upload_Failure_Resolution_Report.md`

The existing investigation identifies the shared photo-upload path and recommends a client-side compression correction before the existing API transfer. However, the persistent-after-failure symptom must also be explicitly tested so that the implementation does not merely hide the underlying retry/state problem.

## 2. Objective

Make Driver photo upload reliable for normal mobile-camera photos and ensure that a failed upload attempt does **not poison the next retry**.

A driver must be able to:

1. select/capture a photo;
2. submit the event;
3. receive success when the upload succeeds;
4. if upload fails, select/capture the same or a new photo and retry without requiring a page refresh;
5. have the retry execute as a genuinely new upload attempt;
6. receive a useful failure state if the underlying upload still fails.

Do not redesign the event workflow.

## 3. Existing Upload Architecture

The current architecture is:

`Event client → uploadPhoto(file, tripId) → POST /api/upload-photo → auth/trip authorization → Supabase Storage event-photos → URL → Event API`

The existing resolution report identifies high-resolution mobile uploads as a likely payload/timeout problem and recommends client-side compression in:

`src/lib/capture/uploadPhoto.ts`

Do not replace the established API/storage/security architecture as part of this implementation.

## 4. EXACT Authorized Application File

Primary authorized implementation file:

`src/lib/capture/uploadPhoto.ts`

If and only if the existing retry/state investigation proves that a calling event client prevents a fresh retry, an additional Driver event client may be changed **only when the implementation report identifies the exact caller and reason**.

Do not broadly refactor all event clients.

## 5. Required Implementation Behavior

### A. Client-side image preparation

Before constructing the existing upload `FormData`, prepare mobile camera images so that unnecessarily large payloads are reduced to a practical size while preserving usable evidence quality.

Requirements:

- perform compression/resizing client-side;
- preserve image orientation as correctly as the browser APIs permit;
- preserve a normal image MIME type supported by the existing upload route;
- preserve visual evidence quality suitable for freight-event proof;
- do not modify the server API contract;
- do not modify Supabase Storage configuration;
- do not modify RLS.

Do not introduce arbitrary aggressive compression that materially damages evidence quality.

### B. Fresh retry semantics

The helper and its callers must not retain a failed upload as a permanently poisoned state.

Verify that:

- each retry creates/uses a valid current upload payload;
- a failed request does not permanently disable future upload attempts;
- stale `File`, `Blob`, promise, error, loading, or completion state is not reused incorrectly;
- a new file selection/capture replaces the previous file correctly;
- the same valid file can be retried when technically safe;
- the upload button/loading state returns to a retryable state after failure;
- a failed upload does not incorrectly mark the event as successfully uploaded;
- no page refresh is required solely to recover from a failed upload.

Do **not** add blind infinite retries.

Do **not** automatically resubmit the event after a failure.

## 6. Error Handling

The current helper may surface only a generic `Failed to upload photo` error.

Improve client-side diagnosability where possible without exposing sensitive server information.

At minimum:

- distinguish an upload HTTP failure from a client-side preparation failure;
- preserve the existing caller-facing failure contract unless a necessary narrow improvement is required;
- ensure loading/error state is reset correctly after every failed attempt;
- do not swallow errors silently.

If the API returns a meaningful status/body, use it for controlled diagnostics rather than replacing it with an unrelated message.

## 7. Important Root-Cause Constraint

The prior resolution report strongly attributes intermittent failure to payload size and/or serverless timeout. That is the implementation basis for compression, but the **persistent retry symptom must not be assumed to be solved by compression alone**.

During implementation/testing, explicitly determine whether the persistent failure is caused by:

- stale `File`/`Blob` state;
- stale promise/request state;
- submit/loading/error state not resetting;
- file input behavior after a failed attempt;
- repeated use of an invalid/consumed payload;
- HTTP 413/504/500 behavior;
- network/request cancellation;
- authentication/session timing;
- another caller-side condition.

If evidence shows that compression fixes the payload issue but a separate retry-state defect remains, stop and report the additional root cause rather than silently expanding into unrelated refactoring.

## 8. Protected Boundaries

**DO NOT CHANGE:**

- database schema;
- migrations;
- RLS policies;
- authentication/authorization rules;
- Supabase Storage bucket configuration;
- event lifecycle semantics;
- event types;
- evidence model;
- event API contract;
- trip ownership/security checks;
- Company portal;
- Reviewer portal;
- Timeline behavior;
- Completed Trips → Timeline historical-selection behavior;
- responsive CSS fixes already completed;
- unrelated Driver UI/UX.

Do not switch to direct client-to-Supabase Storage upload because that would require a separate security/RLS design decision.

## 9. Implementation Rules

1. Inspect the current `uploadPhoto.ts` before editing.
2. Inspect representative Driver event callers when necessary to verify retry-state behavior.
3. Make the smallest safe change that addresses the proven failure mechanism.
4. Reuse the existing upload API.
5. Do not add a new backend endpoint.
6. Do not change the API request contract.
7. Do not add blind retry loops.
8. Do not add arbitrary file rejection rules unless required and justified by evidence.
9. Do not silently modify unrelated files.
10. Preserve successful existing photo-upload behavior.
11. Preserve mandatory-photo workflow behavior.
12. Preserve event submission ordering and existing GPS/timestamp behavior.

## 10. Required Verification

### Automated

After implementation:

1. Run the normal project build.
2. Run TypeScript/type-checking if configured.
3. Confirm no new lint/build errors.
4. Confirm the final diff contains only the authorized implementation files.

### Upload test matrix

Test on a narrow mobile-like viewport and with realistic camera photos:

| Scenario | Required result |
|---|---|
| Normal/smaller photo → first upload | Upload succeeds |
| Larger mobile photo → first upload | Upload succeeds after client preparation, when within supported limits |
| First upload intentionally fails → retry same photo | Retry is genuinely attempted and can succeed; no refresh required |
| First upload fails → select/capture new photo → retry | New photo uploads successfully when underlying services are available |
| Successful upload → event submission | Existing workflow remains correct |
| Failed upload → retry repeatedly | Each attempt returns to a clean retryable state |
| Refresh after failure | Not required for recovery |

### Network evidence

For at least one successful attempt and one failure/retry sequence, inspect browser Network information where available and record:

- request status;
- request duration;
- request payload size where available;
- whether the retry creates a new request;
- response body/status;
- whether the request is cancelled or times out.

The implementation report must distinguish **observed evidence** from assumptions.

## 11. Regression Verification

Verify that:

- Arrival photo upload still works;
- at least one other Driver event photo upload still works;
- uploaded photo URL is still passed to the existing event submission flow;
- existing event success state remains unchanged;
- no responsive-photo behavior is altered;
- Timeline and Arrival Recorded known-good responsive states remain untouched.

## 12. Failure / Stop Conditions

STOP and report instead of expanding scope if:

- compression requires a backend/API change;
- direct Storage upload appears necessary;
- RLS changes appear necessary;
- the existing API contract must change;
- the failure cannot be reproduced or diagnosed sufficiently;
- a retry bug requires broad event-client refactoring;
- authentication/session behavior is identified as the root cause;
- another protected system must change.

A new investigation/decision is required before crossing any protected boundary.

## 13. Required Implementation Report

After implementation, create an implementation report recording:

1. exact root cause(s) confirmed;
2. observed versus inferred failure mechanisms;
3. exact files changed;
4. compression/preparation behavior added;
5. retry-state behavior verified;
6. successful first-upload result;
7. failed-upload → retry result;
8. failed-upload → new-photo retry result;
9. Network evidence/status where captured;
10. build/type-check result;
11. regression results;
12. confirmation that no API/DB/RLS/auth/Storage/security/workflow boundaries changed;
13. commit/reference information.

Then **STOP for Ayush's manual verification/acceptance**.

## 14. Final Handoff Instruction

Implement only the narrowly scoped Driver photo-upload reliability correction described in this prompt.

The critical acceptance criterion is not merely “large photos upload.” It is:

> **If one photo upload attempt fails, the Driver must be able to retry without refreshing the page, and the retry must be a clean, real upload attempt rather than a poisoned/stale continuation of the failed attempt.**

Preserve the existing backend/API/security architecture. Do not make unrelated changes.
