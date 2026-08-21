# Chat4_Node3_Instruction_Day2_GPS_Timestamp_PhotoUpload.md

**Project:** Freight — AI Builders Hackathon
**Chat:** #4 | **Node:** 3 (Build execution) | **Day:** 2 of 25
**Type:** Implementation instruction (build, not a fix)
**For:** Antigravity

---

## Scope (LOCKED — do not exceed)

Build the **infra-level utilities** for event capture. Do NOT wire these into the Arrival/Check-in/Departure event UI yet — that's Day 3. This is Day 2 only:

1. GPS capture utility
2. Server timestamp helper
3. Photo upload utility (client capture → Supabase Storage)

No event page, no event API route, no DB insert logic yet. Just the three reusable pieces, isolated and independently testable.

---

## 1. GPS capture utility

Create a client-side utility (e.g. `src/lib/capture/getGpsLocation.ts`) that wraps `navigator.geolocation.getCurrentPosition`.

- Return a typed result: `{ latitude, longitude, accuracy }` on success.
- Handle and surface these failure cases distinctly (don't collapse into one generic error): permission denied, position unavailable, timeout.
- Reasonable timeout (e.g. 10s) and `enableHighAccuracy: true`.
- Must work in a browser-only context (guard against SSR execution — Next.js will try to run this server-side if not careful).

## 2. Server timestamp helper

Timestamps must come from the **server**, never the client device clock (client clocks can't be trusted for evidence integrity — this is a core product principle, not optional).

- Add a minimal API route (e.g. `src/app/api/server-time/route.ts`) that returns the current server time in ISO 8601 UTC.
- Add a small client helper that calls this route and returns the timestamp string.
- Keep this route unauthenticated and dumb — it does nothing but return `new Date().toISOString()`. No DB, no auth check.

## 3. Photo upload utility

Client capture via `<input type="file" accept="image/*" capture="camera">`, upload to Supabase Storage.

- Upload must go through a Next.js API route using the `service_role` key server-side (per locked write pattern — no direct client-to-Supabase-Storage upload with anon key).
- Bucket: use the existing `test-photos` bucket from Node 2.5 testing, or confirm with Ayush if a new `event-photos` bucket should be created for this project's actual data (do not silently reuse the test bucket for real data without asking).
- API route accepts the image file (multipart/form-data or base64 — Antigravity's choice, whichever is cleaner with the Next.js App Router setup already in place), uploads to Storage, returns the public/storage URL.
- Client utility (e.g. `src/lib/capture/uploadPhoto.ts`) wraps the fetch call to this route and returns the resulting URL.

---

## Test page (for Ayush's manual verification only — not a permanent app page)

Add one throwaway test page (e.g. `src/app/test-day2/page.tsx`) that:
- Has a button to capture GPS and display the result on screen
- Has a button to fetch server timestamp and display it
- Has a file input to pick/capture a photo, upload it, and display the returned URL as an `<img>`

This page is temporary scaffolding for Day 2 verification. Do not link it from any real navigation. It can be deleted once Day 3 wires these into the actual event flow.

---

## Out of scope (do not touch)

- Arrival/Check-in/Departure event pages or API routes
- Any DB insert logic for events
- Timeline view
- AI evidence summary

---

## After implementation

Report back (as a file, per workflow — not pasted in chat):
- Files created/modified
- Build result (`npm run build` or dev server clean start — no errors)
- Any deviation from this instruction and why
- Save to `03_IMPLEMENTATION/implementation_reports/Chat4_Node3_Report_Day2_GPS_Timestamp_PhotoUpload.md`

Ayush will do manual browser verification on the test page after this report — do not mark Day 2 as done until that manual check happens (per evidence rule).

---

## Reminder (per locked write pattern, Section 3 of general-project-setup)

All writes (photo upload here) go through Next.js API routes with `service_role` key, server-side only. Never expose service_role via `NEXT_PUBLIC_*`. RLS stays enabled on all tables/buckets as defense-in-depth even though writes route through server-role.
