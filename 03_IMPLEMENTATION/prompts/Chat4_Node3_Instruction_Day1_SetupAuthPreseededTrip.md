# Chat4_Node3_Instruction_Day1_SetupAuthPreseededTrip.md

**Project:** Freight — AI Builders Hackathon
**Chat:** #4 | **Node:** 3 (Build Execution) | **Day:** 1 of 25 (Aug 21, 2026)
**Assigned to:** Antigravity
**Task:** Core MVP Item #1 — Next.js project setup + Supabase client config + driver-only login + pre-seeded trip in DB

---

## Scope for this instruction (Day 1 ONLY)

1. Next.js project initialization
2. Supabase client configuration — BOTH clients:
   - `lib/supabase-client.ts` — anon key, browser-safe, for READS only
   - `lib/supabase-server.ts` — service_role key, server-only, for WRITES (per verified pattern from Node 2.5)
3. Driver-only login (simple driver ID based auth — no company/multi-role yet, that's Stretch #7)
4. One pre-seeded trip in the DB (manually inserted test trip so Day 2+ event capture has something to attach to)

**Explicitly OUT of scope for Day 1:** event capture (GPS/photo/timestamp), timeline view, AI summary, any stretch item. Do not build ahead.

---

## 1. Next.js Project Setup

- Use `src/` directory convention (confirmed convention for this project)
- TypeScript enabled
- App Router (not Pages Router)
- Tailwind CSS for styling (lightweight, hackathon speed)
- Install `@supabase/supabase-js`

```bash
npx create-next-app@latest freight --typescript --tailwind --app --src-dir
cd freight
npm install @supabase/supabase-js
```

## 2. Supabase Client Configuration

**Active Supabase project (confirmed, Node 2.5):**
- Project name: `freight_hackathon`
- URL: `https://nzsexdmcvhoqsywxxnpe.supabase.co`
- Region: ap-south-1 (Mumbai)

⚠️ Do NOT use the old abandoned project (`freight`, ref `jlxwboeyxzfazvykaost`) — confirmed dead due to platform-side RLS bug.

### `src/lib/supabase-client.ts` (anon key — browser-safe, reads only)
```ts
import { createClient } from '@supabase/supabase-js'

export const supabaseClient = createClient(
  process.env.NEXT_PUBLIC_SUPABASE_URL!,
  process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!
)
```

### `src/lib/supabase-server.ts` (service_role key — server-only, writes)
```ts
import { createClient } from '@supabase/supabase-js'

export const supabaseServer = createClient(
  process.env.SUPABASE_URL!,
  process.env.SUPABASE_SERVICE_ROLE_KEY!
)
```

### `.env.local` (gitignored — do not commit)
```
NEXT_PUBLIC_SUPABASE_URL=https://nzsexdmcvhoqsywxxnpe.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=<from Settings -> API Keys -> anon/public>
SUPABASE_URL=https://nzsexdmcvhoqsywxxnpe.supabase.co
SUPABASE_SERVICE_ROLE_KEY=<from Settings -> API Keys -> service_role>
```

⚠️ Confirm `.env.local` is in `.gitignore` before first commit.

## 3. Driver-Only Login

Simple, no multi-role complexity (company role is Stretch #7, later).

- `drivers` table: `id (uuid, pk)`, `driver_code (text, unique)`, `name (text)`, `created_at (timestamptz, default now())`
- Login = driver enters their `driver_code` → looked up server-side via `supabase-server` → session/local state stores driver id
- No password for MVP (hackathon speed decision — flag this as a known simplification, not a bug)
- Simple `/login` page: input field for driver code → POST to `/api/auth/login` → validates against `drivers` table → returns driver id + name

## 4. Pre-Seeded Trip

- `trips` table: `id (uuid, pk)`, `driver_id (uuid, fk -> drivers.id)`, `facility_name (text)`, `status (text, default 'active')`, `created_at (timestamptz, default now())`
- Manually insert ONE test driver + ONE test trip via Supabase SQL editor (this is a manual DB change — must be captured in a committed migration file, not left as an undocumented manual edit, per workflow rule 10.2)
- Save the exact SQL used into `src/db/seed.sql` in the repo (source of truth for this manual step)

Example seed (adjust values as needed, keep exact SQL used in `seed.sql`):
```sql
insert into drivers (driver_code, name) values ('DRV001', 'Test Driver') returning id;
-- use returned id below
insert into trips (driver_id, facility_name, status) values ('<driver_id>', 'Test Facility', 'active');
```

## 5. RLS

- Enable RLS on both `drivers` and `trips` tables (defense-in-depth, per Node 2.5 decision)
- No client-side INSERT/UPDATE/DELETE policies needed yet — all writes go through service-role server routes only (per Section 4 of Node 2.5 handoff)

## 6. Deliverable / Definition of Done for Day 1

- [ ] Next.js project runs locally (`npm run dev`)
- [ ] Both Supabase clients configured, env vars working
- [ ] `/login` page functional — driver code → validated → session established
- [ ] `drivers` + `trips` tables exist in Supabase with RLS enabled
- [ ] One test driver + one test trip seeded, `seed.sql` committed to repo
- [ ] No event-capture, timeline, or AI features touched

## 7. Reporting back

After implementation, Antigravity should report (for `04_TESTING/test_results/` or `03_IMPLEMENTATION/implementation_reports/` as appropriate):
- Files created/changed
- Build/compile status
- Any deviation from this spec and why
- Confirmation `.env.local` is gitignored
- Confirmation of manual DB seed SQL being committed to `seed.sql`

Ayush will do manual browser login test once Antigravity confirms build is green.

---

**Investigation/fix separation note:** N/A for this instruction — this is fresh build, not a bugfix.
**Evidence expected:** terminal output (build success), screenshot of working `/login` page (Ayush, manual).
