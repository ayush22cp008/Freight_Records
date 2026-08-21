# CHANGELOG.md

## Aug 21, 2026 — Day 1 (Node 3 build execution)
- Node 2 (build plan) revised: 4-day scope superseded by 25-day scope (see ROADMAP.md for full day-by-day schedule)
- Day 1 implementation instruction issued (Chat #4): Next.js setup, Supabase client config, driver-only login, pre-seeded trip
- Bug found + fixed: `drivers` and `trips` tables were never created in Supabase (`relation "drivers" does not exist`) — root cause was a missing CREATE TABLE migration step in original Day 1 spec. Fixed via `src/db/migrations/001_create_core_tables.sql`, committed to repo.
- Day 1 verified complete: driver login flow working end-to-end (valid code → session → protected route; invalid code → proper error, no crash), seeded driver + trip confirmed in DB via Supabase Table Editor and browser manual test.
- ✅ Day 1 / Core MVP Item #1 — LOCKED

## Aug 19, 2026
- Placeholder — project records structure initialized per general-project-setup skill.
