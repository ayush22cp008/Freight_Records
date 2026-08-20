# MASTER_ARCHITECTURE.md

## Product (LOCKED)
Evidence notary app — not a detention calculator. Driver logs 3 events at a freight facility: Arrival → Check-in → Departure. Each event captures GPS + server timestamp + photo (mandatory at Arrival + Departure, optional at Check-in). All records immutable (insert-only). Single AI evidence summary generated after all events — factual only, no AI decisions/blame.

## Stack (LOCKED)
Next.js + Supabase (DB + Auth + Storage) + Vercel. Mobile-responsive web app. NO native app, NO Expo, NO background GPS. Directory convention: `src/` (e.g. `src/app/api/...`, `src/lib/...`).

## Supabase — active project
- Project name: `freight_hackathon`
- URL: `https://nzsexdmcvhoqsywxxnpe.supabase.co`
- Region: South Asia (Mumbai), ap-south-1
- Compute: Nano
- ⚠️ Old project (`freight`, ref `jlxwboeyxzfazvykaost`) is abandoned — do not use.

## Critical pattern: server-side inserts only (LOCKED)
Client-side anon-key INSERT hit a confirmed Supabase platform-level bug (401/42501 RLS violation) — reproduced even on a fresh project, proving it's account/org-level, not config-level.

**Fix (mandatory for every table going forward):** all writes go through a Next.js API route using the `service_role` key server-side only (never exposed to browser, never `NEXT_PUBLIC_*`). RLS stays enabled on all tables as defense-in-depth.

`lib/supabase-server.ts`:
```ts
import { createClient } from '@supabase/supabase-js'

export const supabaseServer = createClient(
  process.env.SUPABASE_URL!,
  process.env.SUPABASE_SERVICE_ROLE_KEY!
)
```

API route pattern (`app/api/<route>/route.ts`):
```ts
import { supabaseServer } from '@/lib/supabase-server'
import { NextResponse } from 'next/server'

export async function POST(req: Request) {
  const body = await req.json()
  const { data, error } = await supabaseServer
    .from('<table_name>')
    .insert({ /* fields */ })
    .select()

  if (error) return NextResponse.json({ error }, { status: 500 })
  return NextResponse.json({ data }, { status: 201 })
}
```

Env vars (server-only, gitignored):
```
SUPABASE_URL=https://nzsexdmcvhoqsywxxnpe.supabase.co
SUPABASE_SERVICE_ROLE_KEY=<Settings → API Keys → service_role>
```
