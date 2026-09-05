# Chat39 — Day 15 — Company Portal Existing Structure Investigation: Shared vs Company-Specific Components Report

## Investigation Findings

This report documents the shared and company-specific frontend components used by the Company portal in the `freight` codebase.

### 1. Shared Components
Components used by the Company and at least one other role:

* **Authenticated Layout**
  * **Name**: `layout.tsx` (AuthenticatedLayout)
  * **Path**: `src/app/(authenticated)/layout.tsx`
  * **Roles**: Used by Company, Driver, and Reviewer.
  * **Behavior Changes**: Handles basic blocking for REJECTED verifications and missing identities globally, but does not alter layout structure based on role.
  * **Evidence**: Wrapping layout for all `(authenticated)` routes.
  * **Confidence**: VERIFIED

* **Navigation / Navbar**
  * **Name**: `Navbar.tsx`
  * **Path**: `src/app/(authenticated)/Navbar.tsx`
  * **Roles**: Used by Company, Driver, and Reviewer.
  * **Behavior Changes**: Does **not** change behavior by role. It universally renders the exact same links ("Dashboard", "Timeline") and the `userEmail` string for everyone.
  * **Evidence**: Code inspection shows no role-checking logic inside `Navbar.tsx`.
  * **Confidence**: VERIFIED

* **Dashboard Page Shell**
  * **Name**: `page.tsx` (Home)
  * **Path**: `src/app/(authenticated)/page.tsx`
  * **Roles**: Used as the root dashboard for both Company and Driver.
  * **Behavior Changes**: Heavily alters behavior based on role. It acts as a routing switch, rendering completely different TSX blocks based on `identity.trusted_role === 'COMPANY'`.
  * **Evidence**: Internal `if (identity.trusted_role === 'COMPANY') { ... }` block handles company presentation, followed by Driver logic.
  * **Confidence**: VERIFIED

### 2. Company-Specific Components
Components and UI blocks exclusive to the Company portal:

* **Company Dashboard UI Blocks**
  * **Path**: Inline inside `src/app/(authenticated)/page.tsx`
  * **Description**: The "Incoming Deliveries", "Completed Deliveries", and "Welcome, Company" blocks are exclusively rendered for companies.

* **Trip Creation Client**
  * **Name**: `CreateTripClient.tsx`
  * **Path**: `src/app/(authenticated)/company/trips/create/CreateTripClient.tsx`
  * **Description**: The multi-step form for creating and publishing a trip.

* **Receiver Check-in Client**
  * **Name**: `ReceiverCheckinClient.tsx`
  * **Path**: `src/app/(authenticated)/company/receiver-checkin/ReceiverCheckinClient.tsx`
  * **Description**: Company workflow for verifying driver arrival at the facility.

* **Receiver Completion Client**
  * **Name**: `ReceiverCompletionClient.tsx`
  * **Path**: `src/app/(authenticated)/company/completion/ReceiverCompletionClient.tsx`
  * **Description**: Company workflow for confirming delivery and unloading of goods.

* **Public Evidence Share Manager**
  * **Name**: `PublicShareManager.tsx`
  * **Path**: `src/app/(authenticated)/company/PublicShareManager.tsx`
  * **Description**: UI for creating, copying, replacing, and revoking public links to evidence.

* **Confidence**: VERIFIED for all above components. They are housed entirely under the `company/` route tree or guarded by strict Company role checks.

### 3. Role-Specific Behavior Assembly
Role-specific behavior is assembled dynamically at the root page level (`src/app/(authenticated)/page.tsx`). The layout (`layout.tsx`) handles global auth requirements (verification blocking), and `page.tsx` injects the correct role-based dashboard view (Company vs. Driver). Reviewers are intercepted even earlier and redirected to `/reviewer/queue`.

### 4. Structural Inconsistencies from Shared Components
**Yes, a major structural inconsistency exists due to `Navbar.tsx`.**
Because `Navbar.tsx` is shared blindly across roles, it renders a "Timeline" link for the Company. However, as discovered in the previous investigation, the `/timeline` route explicitly blocks Companies by forcing a redirect back to `/` if a `driver` record is not found. 
This results in a dead, confusing navigation link for Company users—a direct consequence of sharing the Navbar without role-based conditional rendering.
**Confidence**: VERIFIED.
