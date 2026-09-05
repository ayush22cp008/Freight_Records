# Chat40 — Day 15 — Node 7 — Phase 1b
# Existing Reviewer System Investigation Report

## 1. Reviewer Entry and Routing

**How a user becomes/identifies as a Reviewer**
- A user is identified as a Reviewer if their `auth.users.id` (auth_id) exists in the `reviewer_authorizations` table.

**Routing and Redirect/Intercept Behavior**
- When a user logs in, they land on the global `src/app/(authenticated)/page.tsx` Dashboard router.
- The router checks `reviewer_authorizations` **first** before checking any Freight Identity rules.
- If a match is found, the user is immediately redirected to `/reviewer/queue`.
- **Navigation Trap (BUG/GAB)**: The `(authenticated)/layout.tsx` wraps the queue with the standard `Navbar`. The Navbar exposes "Dashboard" (`/`) and "Timeline" (`/timeline`). 
  - Clicking "Dashboard" hits the router and bounces the reviewer immediately back to `/reviewer/queue`.
  - Clicking "Timeline" hits the driver-only check, bounces to `/`, which bounces back to `/reviewer/queue`. 
  - The Reviewer is effectively trapped on the Queue page with no distinct navigation mechanism other than the sign-out button.

## 2. Reviewer Frontend Surface

The Reviewer system consists of exactly **one page and one component**.

### Surface 1: `src/app/(authenticated)/reviewer/queue/page.tsx`
- **Purpose**: Displays the list of pending applications.
- **User-visible information**: Email, Requested Role, Evidence Type (e.g., Drivers License), Mime Type.
- **Data Source**: Fetches `freight_identities` where `verification_status = 'PENDING'` and manually joins `onboarding_evidence` where `status = 'PENDING'` on the `auth_id`.
- **Loading State**: Handled natively by Next.js server components, no specific skeleton.
- **Empty State**: Renders "No pending applications." inside a card.
- **Responsive Behavior**: Uses `max-w-4xl mx-auto space-y-6`. Displays nicely on mobile because cards stack vertically.

### Surface 2: `src/app/(authenticated)/reviewer/queue/ReviewAction.tsx`
- **Purpose**: Client component handling the approve/reject actions and evidence viewing.
- **User Actions**:
  - **View Evidence Document**: Calls Supabase to generate a 60-second signed URL for the protected storage bucket, replacing the button with a temporary "Open Document" anchor link.
  - **Approve**: Submits the approval to the API.
  - **Reject**: Triggers a native browser `prompt()` asking for a rejection reason, then submits to the API.
- **Loading State**: Disables buttons and changes text to "Processing...".
- **Error State**: Renders inline red text (`{error}`).
- **Success State**: Uses `router.refresh()` to reload the server component (queue page), removing the handled item from the list without a specific success toast.

## 3. Reviewer Workflow / Interaction Trace

The actual supported workflow is incredibly narrow:
1. Reviewer logs in and is forced to `/reviewer/queue`.
2. Reviewer views the list of pending applications.
3. Reviewer clicks "View Evidence Document" to open the uploaded image/PDF in a new tab.
4. Reviewer clicks "Approve" (instantly processing) OR "Reject" (prompted for reason, then processing).
5. The queue refreshes.

**What is explicitly NOT PRESENT:**
- There is no UI to view approved or rejected applications (history).
- There is no UI to view or review trips, deliveries, or operational evidence (timeline events, photos, etc.). The Reviewer is strictly an "Onboarding Identity Reviewer".
- There is no detailed view page (everything is inline on the queue card).
- There is no status/dashboard metrics overview.

## 4. Backend and API Trace

### Endpoint: `POST /api/admin/review`

**Authentication & Authorization:**
- Validates the Supabase session.
- Queries `reviewer_authorizations` for the caller's `auth_id`. Returns 403 Forbidden if not found.

**Data Operations:**
- Validates the identity exists and is `PENDING`.
- **On REJECT**:
  - Updates `onboarding_evidence` to `REJECTED` and sets the `rejection_reason`.
  - Updates `freight_identities` `verification_status` to `REJECTED`.
- **On APPROVE**:
  - Updates `onboarding_evidence` to `APPROVED`.
  - Updates `freight_identities` to `VERIFIED` and sets `trusted_role` to the requested role.
  - **Auto-Provisioning**: Automatically creates the business record:
    - If `DRIVER`: Inserts into `drivers` table, generating a mock `driver_code` (`DRV-` + first 6 chars of identity ID) and setting the name from the email prefix.
    - If `COMPANY`: Inserts into `companies` table, setting the name from the email prefix.

**Response Shape:**
- Consumed directly by `ReviewAction.tsx` which simply checks `!res.ok`. Returns `{ success: true, status: 'VERIFIED' | 'REJECTED' }`.
