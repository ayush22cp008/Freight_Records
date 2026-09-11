# Chat49 — Day 21 — Node 7
## Event-Driven Scoped Auto-Refresh Architecture Investigation Report

**Status:** INVESTIGATION COMPLETE
**Author:** Antigravity

### 1. OBSERVATION: Current Baseline vs. Event-Driven Alternative
Previous investigations established that a global, timer-based `<GlobalAutoRefresh />` utilizing `useRouter().refresh()` is the safest immediate solution to preserve the Server Component architecture. However, timer-based polling incurs constant database read overhead regardless of system activity. 
This investigation explores an **Event-Driven Scoped Auto-Refresh** architecture using Supabase Realtime subscriptions to trigger `router.refresh()` only when relevant state changes occur in the database.

### 2. INVESTIGATION: Feasibility of Supabase Realtime + Next.js App Router
- **How it works:** A lightweight Client Component (e.g., `<RealtimeRefresher />`) is mounted. Instead of `setInterval`, it uses `@supabase/supabase-js` to listen for Postgres changes (INSERT, UPDATE, DELETE) on specific tables. When an event fires, it calls `router.refresh()`.
- **Scope Alignment:** 
  - The **Reviewer Queue** could listen to `freight_identities` and `onboarding_evidence`.
  - The **Company Dashboard** could listen to `trips` and `receiver_delivery_requests` filtered by their `company_id`.
  - The **Driver Dashboard** could listen to `trips` filtered by their `driver_id`.

### 3. EVIDENCE: Risk Profile and Architectural Constraints
While event-driven architectures reduce idle database load, introducing Supabase Realtime presents specific constraints in this locked baseline:

- **Constraint 1 (RLS and Realtime):** By default, Supabase Realtime respects Row Level Security (RLS). However, setting up client-side subscriptions requires exposing the `supabase-browser-client` and establishing websocket connections. The current dashboards heavily rely on server-side `supabaseServer` fetching.
- **Constraint 2 (Table Configuration):** Supabase Realtime is disabled on tables by default. It must be explicitly enabled via SQL `alter publication supabase_realtime add table ...`. This requires a database migration.
- **Constraint 3 (Complex Component Scoping):** To prevent unnecessary global refreshes, the Realtime listener cannot be placed at the root layout. It must be scoped at the page level, meaning `page.tsx` for Driver/Company and `reviewer/queue/page.tsx` would need companion client components injected into them.

### 4. ROOT CAUSE & ARCHITECTURE DECISION
**Pros of Event-Driven Scoped Refresh:**
- Zero idle database load; updates only trigger when data mutates.
- Near-instantaneous UI updates (no 30-second stale window).

**Cons of Event-Driven Scoped Refresh:**
- Requires database schema modifications (enabling Realtime publications).
- Requires instantiating the Supabase client-side library (`createBrowserClient`) purely for websocket event listening, increasing client bundle size.
- Higher complexity in managing subscription lifecycle and avoiding memory leaks on route changes.

### 5. DECISION & RECOMMENDATION
**Recommendation:** Do **not** adopt Event-Driven Scoped Auto-Refresh for Phase 1 MVP unless the 30-second polling load is proven to be a hard bottleneck.

The complexity of configuring Postgres publications, injecting Supabase client-side websocket listeners, and managing component lifecycles introduces unnecessary risk to the currently stable Server Component architecture. The previously approved timer-based soft refresh (`setInterval` + `router.refresh()`) achieves the same product requirement with significantly less architectural friction.

**Next Steps:**
Awaiting your decision on whether to proceed with the simple Global Poller (as recommended previously) or if the Realtime overhead is strictly required for this project phase.
