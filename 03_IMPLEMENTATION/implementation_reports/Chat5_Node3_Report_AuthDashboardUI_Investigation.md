# Chat5 Node3 — Investigation Report: UI Architecture, Navbar & Dashboard

## Objective
Provide a factual audit of the current UI and application structure, evaluating the current Dashboard, Login UI, and the viability of adding a shared Navbar and transitioning to Email/Password authentication.

---

## 1. Repository & Application Structure
- **App Structure:** A standard Next.js App Router structure in `src/app`.
- **Layouts:** Only a single `RootLayout` exists (`src/app/layout.tsx`). It contains only boilerplate HTML/Body/Font setup. There are no nested layouts or route groups.
- **Shared Components:** **None.** There is no `src/components` or `src/ui` directory. All UI elements are standard HTML nodes styled with inline Tailwind CSS.
- **Navbar:** **None.** The application currently lacks a shared Navbar or application shell. Each page renders its own isolated UI within the `<body>`.
- **Styling:** Standard Tailwind CSS classes used directly on elements.

---

## 2. Current Login UI (`src/app/login/page.tsx`)
- **Structure & Fields:** A centered white card on a light gray background. It contains a single text `<input>` for "Driver Code" and a single Submit `<button>`.
- **Behavior:** Controlled component using React `useState`. Prevents default form submission, sets a loading state, calls `fetch('/api/auth/login')`, handles errors inline, and redirects to `/` on success.
- **Components:** Pure HTML/Tailwind. No external or internal abstracted UI components are used.
- **Adaptability to Email/Password:** **Extremely adaptable.** Because it relies on basic standard HTML inputs rather than rigid pre-built components, it can be trivially updated to an Email/Password form by adding an extra `<input>` field and updating the API payload.

---

## 3. Current Authentication Foundation
- **Session Mechanism:** Uses a custom `driver_id` HttpOnly cookie set by `src/app/api/auth/login/route.ts`. 
- **Supabase Auth:** **Not currently used.** The login API simply does a raw SQL query against the `drivers` table (`driver_code` column).
- **Required Changes for Email/Password:**
  1. **Database:** Either integrate official Supabase Auth (creating users in `auth.users`) OR add a password column to the custom `drivers` table.
  2. **API:** Update `/api/auth/login` to validate against the new auth method. If using Supabase Auth, transition from custom cookies to standard Supabase Auth cookies (`@supabase/ssr`).
  3. **Protected Routes:** Update the server components (`app/page.tsx`, `events/arrival/page.tsx`) to verify the new Supabase Auth session instead of checking for the custom `driver_id` cookie.

---

## 4. Current Dashboard / Hub (`src/app/page.tsx`)
- **Structure:** Server Component that fetches data directly. Renders a simple centered container with a header ("Active Trip: [Name]") and a white card indicating the "Current Status" and a full-width blue CTA button.
- **Trip Information:** Currently only fetches and displays `facility_name`.
- **Progress & Next Actions:** Hardcoded logical checks verify the existence of `arrival`, `checkin`, and `departure` events in the database to conditionally render the single primary CTA (e.g., "Start Check-in" which links to `/events/checkin`).
- **Integration with a Navbar:** Currently acts as the absolute root UI. If a shared Navbar (with "Dashboard", "Active Trip", "Timeline", "Sign out") is introduced, it should be implemented in a new layout (e.g. `src/app/(authenticated)/layout.tsx`) so the Hub can cleanly sit inside it without redundant code.
