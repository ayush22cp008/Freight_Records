# Chat39 — Day 15 — Company Portal Existing Structure Investigation Continuation Report

## Investigation Findings (Part 2)

This report continues the investigation of the existing frontend structure for the Company Portal in the `freight` codebase.

### 1. Evidence & Public Sharing
- **Source Paths**: Implemented in `src/app/(authenticated)/company/PublicShareManager.tsx` and used within `src/app/(authenticated)/page.tsx`.
- **Where the Company sees evidence**: The Company does *not* see internal proof/evidence photos directly on their dashboard or in a timeline. Instead, they can generate a Public Share URL.
- **How PublicShareManager is used**: The component is rendered for each trip in the "Completed Deliveries" list. It allows the company to hit `/api/trips/:tripId/public-share` to "Create Public Share", "Replace Share", or "Revoke Share".
- **Availability**: Sharing is only available for **Completed Trips**. Active/Incoming deliveries do not have the PublicShareManager rendered. 
- **Confidence**: VERIFIED (Directly evident in `page.tsx`).

### 2. Completed Trips / History / Timeline
- **Company Experience for Completed Deliveries**: They are listed at the bottom of the Company Dashboard (`/`) with basic info (Pickup, Dropoff, "Completed" badge, and the PublicShareManager). 
- **Timeline/History Access**: **The Company cannot access the Timeline.**
  - The global `/timeline` route (`src/app/(authenticated)/timeline/page.tsx`) explicitly attempts to resolve the authenticated user to a `driver` record (`.from('drivers').eq('auth_id', user.id)`). 
  - If a driver record is not found (which is true for standard Company users), the route forcefully redirects to `/`.
  - Therefore, the Timeline is strictly Driver-specific (and potentially Reviewer-specific in other contexts), but inaccessible to the Company role.
- **Confidence**: VERIFIED (Directly evident in `timeline/page.tsx` redirect logic).

### 3. Profile / Account
- **Profile UI**: **NOT FOUND**. There is absolutely no dedicated Company profile, account settings, or profile editing UI implemented in the source code.
- **Current State**: The company's identity is managed via backend row-level mapping (`auth_identities` and `companies`), and the only surface exposing account info is the global Navbar showing the `user.email`.

### 4. Responsive / Mobile Structure
- **Company Dashboard (`page.tsx`)**: Uses standard Tailwind mobile-first responsive utilities. 
  - Incoming trips stack vertically on mobile and row-align on larger screens: `flex flex-col sm:flex-row justify-between sm:items-center`.
  - The container uses `max-w-4xl mx-auto` to center content on wide screens.
- **Trip Creation (`CreateTripClient.tsx`)**: The form uses `space-y-4` for vertical stacking, but statically forces `grid grid-cols-2` for the Distance/Duration fields without responsive breakpoints (meaning it will remain side-by-side even on small mobile screens).
- **Confidence**: VERIFIED (Directly evident in the Tailwind class names applied).
