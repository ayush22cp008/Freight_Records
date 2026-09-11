# Chat49 — Day 21 — Node 7
## Cross-Portal Global Auto-Refresh Investigation Report

**Status:** INVESTIGATION COMPLETE
**Author:** Antigravity

### 1. OBSERVATION: Global Data Fetching Implementation
I investigated the potential for a **Global Auto-Refresh** mechanism across all three operational portals:
- **Driver Dashboard** (`src/app/(authenticated)/page.tsx`)
- **Company Dashboard** (`src/app/(authenticated)/page.tsx`)
- **Reviewer Queue** (`src/app/(authenticated)/reviewer/queue/page.tsx`)

Currently, all data fetches are performed strictly via Server-Side Rendering (SSR) in React Server Components using `supabaseServer`. There is no global polling, realtime event bus, or client-side caching (e.g. SWR/React Query) in place to automatically reflect data changes globally without a user-initiated hard navigation.

### 2. INVESTIGATION: When Fetches Occur & Global State Synchronization
- **When fetches occur:** Only during initial page load, or full/soft navigation events.
- **Why global sync is challenging:** Because the portals are implemented as server components, real-time sync across different browser sessions (e.g., Company updating a trip, Reviewer approving an applicant) requires a mechanism that can invalidate the Server Component cache from the client-side seamlessly, across the entire application footprint.

### 3. EVIDENCE: Cross-Portal State Changes Requiring Global Sync
To maintain accurate state across all actors in the platform:
- **Company ↔ Driver:** When a driver completes a delivery, the company dashboard must reflect the "Attention Needed" state.
- **Reviewer ↔ Driver/Company:** When a reviewer approves an identity, the onboarding guard must be lifted globally.
- **Global Constraints:** Any solution must not disrupt the locked React Server Component baseline architecture or the existing RLS policies.

### 4. ROOT CAUSE & ARCHITECTURE CONSTRAINTS
Transforming the architecture to support native Supabase Realtime subscriptions globally would require converting page-level components into Client Components, significantly altering the application's lifecycle, persistence semantics, and locked UI behavior. This introduces a major risk to the established project baseline.

### 5. DECISION & RECOMMENDATION
**Can a global auto-refresh be added safely?** Yes, by adopting a non-invasive, timer-based global router refresh.

**Recommended Safest Mechanism (Global Poller Wrapper):** 
Implement a lightweight, global Client Component (e.g., `<GlobalAutoRefresh interval={30000} />`) that wraps the authenticated layout (`src/app/(authenticated)/layout.tsx`). This component will call `useRouter().refresh()` on a set interval (e.g., every 30 seconds).

**Benefits of this approach:**
- **Zero API contract changes:** Preserves the existing `supabaseServer` fetching logic.
- **Global coverage:** By placing it in the layout, all authenticated dashboards automatically receive soft-refresh capabilities without rewriting individual pages.
- **State preservation:** Next.js `router.refresh()` fetches the updated Server Component payload without destroying local client state (like scroll position or open modals).

**Identified Risks:**
- **Database Load:** A global 30-second polling interval for all active users will increase database reads.
- **UX Stale Windows:** Data may be up to 30 seconds out-of-sync before the next tick.

**Next Steps:**
Awaiting explicit approval to inject `<GlobalAutoRefresh />` into the authenticated layout to achieve global synchronization safely.
