# Chat49 — Day 21 — Node 7
## Cross-Portal Auto-Refresh Investigation Report

**Status:** INVESTIGATION COMPLETE
**Author:** Antigravity

### 1. OBSERVATION: Current Data Fetching Implementation
I inspected the current source implementation across the three portals (Driver, Company, Reviewer).
- **Driver and Company Dashboards** (`src/app/(authenticated)/page.tsx`): Fetch data for active trips, completed trips, and incoming receiver requests directly in the React Server Component using `supabaseServer`.
- **Reviewer Queue** (`src/app/(authenticated)/reviewer/queue/page.tsx`): Fetches pending identities and evidence directly in the React Server Component using `supabaseServer`.

### 2. INVESTIGATION: When Fetches Occur & Refresh Mechanisms
- **When fetches occur:** Fetches only happen during server-side rendering (SSR) on initial page load or when standard Next.js navigation occurs.
- **Existing mechanisms:** The application currently relies purely on SSR. There are **no** client-side polling mechanisms, Supabase Realtime subscriptions, React Query, or SWR implementations present for these dashboard state fetches. Data is only invalidated/refreshed upon hard reload or navigation.

### 3. EVIDENCE: Cross-Portal State Changes
State changes that should ideally be visible automatically include:
- **Company Portal:** New incoming delivery requests (`receiver_delivery_requests`), updates to active created trips, and trips needing attention (e.g., driver arrived).
- **Reviewer Portal:** New user registrations and submitted onboarding evidence.
- **Driver Portal:** Status changes of an active trip (e.g., rejected delivery or cancelled trip).

### 4. ROOT CAUSE & ARCHITECTURE CONSTRAINTS
The primary constraint is that the portals are built as locked **Server Components**. 
- Converting these dashboards to Client Components to use `useQuery`, `SWR`, or `Supabase Realtime` would require a significant rewrite of the business rules, data fetching logic, and authorization flow (RLS), fundamentally altering the locked portal product behavior.
- Next.js App Router allows client-side router refreshes (`useRouter().refresh()`) which softly re-fetches the Server Component payload without losing client state.

### 5. DECISION & RECOMMENDATION
**Can auto-refresh be added safely?** Yes, but only if we preserve the Server Component architecture.

**Recommended Safest Mechanism:** 
Introduce a small, dedicated Client Component (e.g., `<AutoRefresh interval={30000} />`) that calls `useRouter().refresh()` on a set interval (e.g., 30 or 60 seconds). This component can be dropped into the existing server-side dashboards without changing any API contracts, RLS, persistence, or evidence semantics.

**Risks to acknowledge:**
- *Stale data:* Data will still be stale between intervals (up to 30s).
- *Performance / Duplicate Requests:* Polling every 30 seconds per active tab will increase database load. We should keep the interval conservative (e.g., 30s-60s) to avoid unnecessary strain.
- *UX Risks:* While `router.refresh()` is seamless, it can occasionally cause slight visual jitter if loading states are triggered unexpectedly. 

**Next Steps:**
Awaiting explicit approval to implement a Client-Side AutoRefresh wrapper for `page.tsx` and `reviewer/queue/page.tsx` using `router.refresh()`.
