# Phase 1b: Driver Frontend Redesign

The objective is to restructure the Driver portal according to the locked `Driver_Locked_Blueprint.md`, breaking down the monolithic `page.tsx` into dedicated, role-aware surfaces without altering backend behavior, APIs, or database schemas.

## Proposed Changes

---

### Shared Layout & Navigation

#### [MODIFY] `Navbar.tsx`
- Update to accept a `role` prop (e.g., `'DRIVER' | 'COMPANY' | 'REVIEWER'`).
- Conditionally render Driver navigation links: 
  - Dashboard (`/`)
  - Available Trips (`/driver/available`)
  - My Active Trip (`/driver/active`)
  - Completed Trips (`/driver/history`)
  - Profile (`/profile`)
- Improve responsive mobile menu layout (hamburger menu) to ensure links are touch-friendly.

#### [MODIFY] `layout.tsx`
- Fetch the user's role (Driver/Company/Reviewer) server-side and pass it down to `Navbar.tsx`.

---

### Driver Dedicated Routes

#### [MODIFY] `page.tsx` (Dashboard)
- Refactor the Driver rendering branch.
- Remove inline lists of trips.
- Add clear navigation call-to-actions to the dedicated routes (My Active Trip, Available Trips, Completed Trips).
- Maintain the Company and Reviewer routing logic as-is.

#### [NEW] `src/app/(authenticated)/driver/available/page.tsx`
- Fetch and display `published` trips where `driver_id` is null.
- If the Driver already has an active trip, render the trips as view-only.
- Implement the 'Available Trips' responsive card layout.
- Link each card to `/driver/trip/[id]`.

#### [NEW] `src/app/(authenticated)/driver/trip/[id]/page.tsx`
- Detailed trip view fetching specific trip information.
- Render the `ClaimTripButton` (Accept Trip) only if the Driver does not currently have an active trip.

#### [NEW] `src/app/(authenticated)/driver/active/page.tsx`
- Fetch the Driver's current active/claimed/in_progress trip.
- Render "No Active Trip" state if none exists.
- Implement the operational hierarchy: Current Status → Next Required Action → Delivery Progress → Evidence Status → Timeline / History.

#### [NEW] `src/app/(authenticated)/driver/history/page.tsx`
- Fetch completed trips for the Driver.
- Render a responsive list/grid.
- Provide a 'View Timeline' action leading to `/timeline?tripId=[id]`.

#### [NEW] `src/app/(authenticated)/profile/page.tsx`
- Fetch and display basic driver identity information (Name, Email).
- Expose the Sign Out action.

---

## Verification Plan

### Automated Tests
- Run `npm run build` to verify no compilation errors are introduced by the new routes.

### Manual Verification
1. **Role Routing:** Log in as a Driver, ensure `Navbar` only shows Driver links.
2. **Dashboard:** Verify Dashboard prioritizes My Active Trip CTA (if active) or Available Trips CTA (if empty).
3. **Available Trips:** Ensure trips can be viewed. If active, Accept Trip is disabled.
4. **Active Trip:** Verify lifecycle buttons (e.g. "Start Arrival", "Record In-Transit") function properly.
5. **Completed Trips:** Ensure history is visible and links to the Timeline.
6. **Responsive Layout:** Check mobile views using Chrome DevTools.
