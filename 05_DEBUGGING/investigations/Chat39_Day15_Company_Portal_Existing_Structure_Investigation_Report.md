# Chat39 — Day 15 — Company Portal Existing Frontend Structure Investigation Report

## Investigation Findings

This report details the existing frontend structure and UI of the Company Portal as currently implemented in the `freight` source code.

### 1. Existing Company routes/pages
- **`/` (Dashboard)**: The root authenticated `page.tsx` renders the Company Dashboard when the user's `trusted_role` is `COMPANY`. It is shared logic with the Driver dashboard, but renders completely different UI blocks for Companies.
- **`/company/trips/create`**: Dedicated route for the company to create and publish new trips.
- **`/company/receiver-checkin`**: Dedicated route for the receiving company to confirm a driver's arrival and check them in.
- **`/company/completion`**: Dedicated route for the receiving company to confirm delivery completion after goods are unloaded.

### 2. Existing Company navigation
- **Navbar**: The company uses the global authenticated `Navbar` (`src/app/(authenticated)/Navbar.tsx`).
- **Links**: The Navbar provides only "Dashboard" and "Timeline" links, along with the user's email and a "Sign out" button.
- **Differences**: There are no role-based differences in the Navbar itself; it is identical to what the Driver sees. There is no sidebar or mobile-specific navigation menu implemented.

### 3. Existing Company dashboard/home
The Company Dashboard (`src/app/(authenticated)/page.tsx`) currently displays:
- **"Incoming Deliveries"**: A list of active/in-progress trips where the `receiving_company_id` matches the authenticated company. It shows the trip status and relevant Call-to-Action (CTA) links (e.g., "Complete Receiver Check-in →").
- **"Completed Deliveries"**: A list of recently completed trips. This section integrates the `PublicShareManager` component to manage public sharing URLs.
- **"Welcome, Company"**: A dedicated card at the bottom containing a "Create New Trip" button linking to `/company/trips/create`.

### 4. Existing trip creation/publishing UI
Implemented in `src/app/(authenticated)/company/trips/create/CreateTripClient.tsx`. It features a 3-step workflow:
1. **Create (Form)**: The user inputs Pickup Facility, Destination Facility, Receiving Company, Distance, Duration, and Payout Offer. Submitting this saves the trip in a "draft" state.
2. **Review (Draft)**: Displays a read-only summary of the entered data and the "draft" status. The user must click "Publish Trip" to proceed.
3. **Published (Success)**: Displays a success message indicating the trip is now available for drivers, along with a link to return to the Dashboard.

### 5. Existing trip tracking / delivery monitoring UI
The Company Dashboard tracks incoming deliveries by calculating a `statusText` based on the sequence of recorded events:
- **"Arrived - Action Required"**: If the driver has arrived but the receiver hasn't checked them in yet (displays CTA to `/company/receiver-checkin`).
- **"Driver is Unloading"**: If the receiver has checked the driver in, but the driver hasn't departed yet.
- **"Action Required"**: If the driver has departed but the receiver hasn't confirmed delivery (displays CTA to `/company/completion`).
- **"Waiting for Driver Confirmation"**: If the receiver confirmed, but the driver hasn't finalized their end.

The Company does not have a dedicated "Live Map" or separate tracking page; tracking is entirely state-text based on the Dashboard list.
