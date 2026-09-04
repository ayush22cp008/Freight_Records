# Chat38 Day14 — Driver Profile Existing Structure Investigation Report

## Investigation Findings

This report details the existing source-code implementation of the Driver Profile / account presentation in the `freight` codebase.

### 1. Dedicated Driver Profile page/route
- **No dedicated profile route exists**. There is no `/profile` or `/driver/profile` page in the `src/app` directory.

### 2. Driver Dashboard/Navbar identity exposure
- The global **Navbar** (`src/app/(authenticated)/Navbar.tsx`) displays the user's email address.
- The **Driver Dashboard** (`src/app/(authenticated)/page.tsx`) displays the driver's name in the header: `Welcome, {driver.name}`.

### 3. Available profile/account data on the frontend
- **Email**: `user.email` (from Auth session).
- **Name**: `driver.name` (from `drivers` table).
- *(Note: Identity metadata such as `verification_status` and `trusted_role` are fetched for routing and authorization, but are not visually presented as "profile information" to the user.)*

### 4. Backend/API/database sources
- **Supabase Auth API**: Provides `user.email` via `supabase.auth.getUser()`.
- **`auth_identities` table**: Fetched via `getFreightIdentity()` for routing logic (`trusted_role`, `verification_status`).
- **`drivers` table**: Fetched in `page.tsx` using `.eq('auth_id', user.id)` to retrieve `id` and `name`.

### 5. Existing profile actions
- The only existing account action is **Sign Out**, located in `Navbar.tsx` (which calls `/api/auth/logout`). 
- There is no functionality implemented to view full details, edit profile, or update settings.

### 6. Role/identity logic impact on profile surfaces
- Because there is no distinct profile surface, there are no role-specific profile changes. The `Navbar` is universally shared across the `(authenticated)` layout and simply renders the email for all roles (Driver, Company, Reviewer).

### 7. Loading, empty, missing-profile, or error states
- **Missing Driver Record Error**: If a user bypasses onboarding but lacks a row in the `drivers` table, `page.tsx` renders a specific error state:
  - *Title*: "No Driver Profile"
  - *Message*: "Your account is not linked to a driver record. Please contact an admin."
- **Missing Identity Error**: If a user lacks an entry in `auth_identities` (and is not a reviewer), `layout.tsx` catches this and displays: "Identity not found. Please contact support."
- **Rejected Identity Error**: If `verification_status === 'REJECTED'`, `layout.tsx` blocks access and shows an "Application Rejected" message.

### 8. Existing responsive behavior
- The `Navbar` applies responsive classes (`hidden sm:ml-6 sm:flex`) which handle the layout of the navigation items, email text, and the Sign out button.
- The "No Driver Profile" error state uses `max-w-2xl mx-auto` to center itself on larger screens.

### 9. Exact source files, routes, components, API calls
- **`src/app/(authenticated)/layout.tsx`**: Renders the global layout, handles `auth_identities` verification blocking, and passes `user.email` to the Navbar.
- **`src/app/(authenticated)/Navbar.tsx`**: The UI component responsible for displaying the email and the Sign Out button. Uses POST `/api/auth/logout`.
- **`src/app/(authenticated)/page.tsx`**: Fetches the `drivers` row (`select('id, name')`) and displays the welcome message if the user is a driver.
