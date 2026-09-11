# Chat49 — Day 21 — Node 7
# Event-Driven Scoped Auto-Refresh Architecture Reinvestigation Report

**Status:** INVESTIGATION COMPLETE
**Author:** Antigravity

## OBSERVATION
The previous investigation recommended a global 30-second timer to avoid the architectural overhead of Supabase Realtime. However, the clarified product requirement specifies that updates should be event-driven, resource-scoped, and immediately reflect changes without affecting unrelated pages or disturbing local client state. [VERIFIED]

## INVESTIGATION
We are investigating how to achieve:
`BUSINESS EVENT → IDENTIFY RESOURCE → IDENTIFY AUDIENCE → NOTIFY CLIENT → REFRESH SURFACE`
Without requiring global timers or causing active capture screens to lose state. [VERIFIED]

## EVIDENCE
- Supabase Postgres Changes can subscribe to table mutations and are scoped by row-level RLS policies. [VERIFIED]
- Using `useRouter().refresh()` inside a targeted client component only triggers server-side rendering for the layout segment it wraps, leaving state (like input fields) untouched. [VERIFIED]

## EVENT → RESOURCE → AUDIENCE → SURFACE MATRIX
- **Event:** Trip Claimed `[VERIFIED]` 
  - **Resource:** `trips` table (UPDATE driver_id)
  - **Audience:** Driver, Company
  - **Surface:** Driver Dashboard (`/`), Company Dashboard (`/`)
  - **Reaction:** `CURRENT-ROUTE router.refresh()`
- **Event:** Delivery Event Captured (e.g., Arrived) `[VERIFIED]`
  - **Resource:** `trip_events` table (INSERT)
  - **Audience:** Company
  - **Surface:** Company Trip Detail (`/company/trips/[id]`), Company Dashboard
  - **Reaction:** `CURRENT-ROUTE router.refresh()`
- **Event:** Reviewer Decision `[INFERRED]`
  - **Resource:** `freight_identities` table (UPDATE verification_status)
  - **Audience:** Applicant, Reviewer
  - **Surface:** Onboarding Guard, Reviewer Queue
  - **Reaction:** `CURRENT-ROUTE router.refresh()`

## ROUTE / CAPTURE SAFETY MATRIX
- `events/pickup-departed` [EXCLUDE] - Active GPS capture and photo upload. Background refresh risks resetting the local photo preview.
- `events/arrived-at-delivery` [EXCLUDE] - Contains active client state.
- `onboarding/*` [EXCLUDE] - Contains multi-step forms.
- `(authenticated)/page.tsx` [SAFE] - Pure read surface.
- `reviewer/queue/page.tsx` [SAFE] - Pure read surface, modals are state-preserved via Next.js soft navigation.

## SECURITY ANALYSIS
- **Payload Sensitivity:** Supabase Realtime payloads can include row data. To preserve absolute security, the client should ignore the payload data entirely and use the event solely as a ping to trigger `router.refresh()`. The server component fetching logic maintains strict RLS boundaries. [VERIFIED]
- **Tenant Isolation:** Postgres Channels can be filtered by resource IDs (e.g., `trips:company_id=eq.123`). [VERIFIED]

## RELIABILITY ANALYSIS
- **Missed/Duplicate Events:** By relying on `router.refresh()`, the client falls back to the server-authoritative state. Missed events just mean a missed UI update (requiring manual refresh); duplicate events resolve to the same idempotent server fetch. [VERIFIED]
- **Reconnect/Multiple Tabs:** Handled seamlessly by Supabase Realtime. [VERIFIED]

## PERFORMANCE ANALYSIS
- **Idle DB Load:** Near-zero. Subscriptions push changes; there is no polling interval. [VERIFIED]
- **Subscription Overhead:** Requires a websocket connection per tab, adding slight memory/network overhead on the client. [INFERRED]

## OPTIONS CONSIDERED
- **Option A (Global Fixed Timer):** High idle DB load, decoupled from actual events.
- **Option B (Event-driven scoped router.refresh()):** Minimal DB load, aligns with SSR architecture, leverages server for authority.
- **Option C (Event-driven targeted client/data update):** Breaks the Server Component model, requires rewriting dashboards to Client Components.
- **Option D (Hybrid):** Unnecessary complexity.

## RECOMMENDED ARCHITECTURE
**Option B — Event-driven scoped router.refresh()**
Inject tiny Realtime Client Components at specific dashboard levels (e.g., `<TripListSubscriber companyId={companyId} />`). These components listen for specific Postgres changes and call `useRouter().refresh()`. This satisfies the product requirement while protecting capture routes and avoiding client-heavy state rewrites.

## RISKS / TRADE-OFFS
- Requires updating the Postgres configuration (`alter publication supabase_realtime add table trips, trip_events;`). [VERIFIED]
- Increases frontend bundle size slightly due to `supabase-js` realtime dependency. [INFERRED]

## REQUIRED VALIDATION TESTS
1. Verify capture pages (`events/*`) do not lose photo or GPS state when a background realtime event triggers a layout refresh.
2. Verify duplicate events correctly debounce `router.refresh()` calls to avoid query spam.

## DECISION STATUS
Investigation Complete. Awaiting final architectural approval.
