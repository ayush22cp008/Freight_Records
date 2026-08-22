# Claude Brain Handoff — Deploy → Authentication Plan Review

**Project:** Freight — AI Builders Hackathon
**Node:** Node 3 — Build Execution
**Brain:** Claude (architecture/reasoning)
**Date:** Aug 22, 2026
**Status:** Hackathon Work Day 2 — IN PROGRESS (not closed)

---

## Context

Day 2 core MVP is implemented and verified ahead of the original 25-day roadmap:
Arrival → Check-in → Departure → Timeline → AI Evidence Summary (Groq-based).
Full detail in `Hackathon_Day1_Day2_Work_Progress_Report.md` (00_PROJECT_CONTROL).

Ayush is now planning the next phase. This file records the plan and Claude's review
of it, for continuity across brains (Claude / ChatGPT / Antigravity).

---

## Ayush's Plan (as stated)

1. **Deploy the current MVP to Vercel first**, and test the full flow live —
   since the team is running ahead of schedule, this is a good checkpoint.
2. **Then build a new authentication system**, replacing the current email/password
   auth, with the following requirements:
   - New driver signup generates a **unique ID** (acts as the login credential,
     like a username — not just a reference number).
   - The unique ID is **emailed to the driver** at signup.
   - Login happens via **unique ID + password** (not email).
   - **One driver = one account**, enforced via **email + device binding**:
     - Same email cannot create a second account.
     - Same device cannot create a second account.
     - A new account requires both a new email AND a new device.
   - **Custom domain** for outgoing email (so the ID email looks professional /
     doesn't land in spam) — same pattern as TableFlow's Resend + custom domain setup.
   - **Account deletion** removes the unique ID from the database permanently —
     it becomes unusable / unreachable by anyone after deletion.
3. Auth implementation will be done **with ChatGPT's help** as the reasoning brain
   for that specific node, following the existing multi-brain workflow.

---

## Claude's Review

### Sequencing — Approved
Deploy-first is the right call. The current auth (Supabase email/password,
`drivers.auth_id` mapping, locked in Chat5) is sufficient to deploy and test the
core evidence-capture flow (the actual hackathon-critical path) on Vercel now.
This surfaces prod-only risks early — env vars, Supabase prod config, Groq
server-side key handling, build differences — while they're still cheap to fix.
The auth redesign is an isolated layer and can safely follow without touching
or re-risking the locked event/timeline/AI flow.

### Auth Redesign — Scoped Correctly, With Known Limits
This is a genuinely new auth system, not a TableFlow copy-paste. Two parts carry
real technical limits that should be documented before ChatGPT starts building,
so expectations are set correctly:

1. **Device binding is inherently soft.** Browsers have no reliable persistent
   device identifier. Practical implementation = fingerprint (canvas/screen/timezone
   hash) or a persistent local token checked against a DB record. Both are
   bypassable (incognito, cleared storage, different browser). This stops casual
   duplicate-account creation, not a determined bypass. Should be documented as a
   known/accepted limitation, not treated as a hard guarantee.
2. **Custom domain email sending** requires SMTP/Resend setup distinct from
   Supabase Auth's default email — same pattern already proven in TableFlow, so
   this is low-risk, just needs to be built explicitly (not a Supabase default).

### Continuity Requirement for the ChatGPT-led Auth Node
Since ChatGPT will be the reasoning brain for this node, the existing project
discipline must still apply so records don't fragment across brains:
- Investigation and fix remain separate steps/prompts.
- Any master prompt / node-map produced by ChatGPT for this node is saved to
  the Drive bridge folder (ChatGPT folder / relevant Master_Prompts path) —
  not left in chat only.
- Instructions to Antigravity still follow the standard file naming and live
  in `02_Instructions/` only.
- GitHub push still requires Ayush's explicit go-ahead, executed by Antigravity.
- Any manual DB/console changes (e.g. new `driver_id` column, unique constraints,
  Resend domain config) get captured into a committed file immediately.

### Verdict
**Plan is safe to proceed.** No conflict with locked Day 1/Day 2 work. Suggested
immediate next step: generate the Vercel deploy instruction for Antigravity,
test live, then open a new Node/Chat for the auth redesign with ChatGPT.

---

## Next Actions
- [ ] Deploy current build to Vercel (Antigravity instruction — pending)
- [ ] Ayush manually verifies full live flow (Arrival → Check-in → Departure → Timeline → AI Summary)
- [ ] Open new node for auth redesign (unique-ID login, device+email binding, domain email, delete-purges-ID)
- [ ] ChatGPT to lead auth node reasoning; Claude available for cross-check if needed
