I'll start by reading the general project setup skill since this references the records-repo/Node-Map system, then pull the actual documents.I don't have GitHub connected as a tool here, so I can't browse the repo directly via web_fetch (blocked by robots) or raw githubusercontent (not pre-seeded). Let me check if a GitHub connector/tool is available.Now let me pull all the required documents.Note: PROJECT_STATE.md appears stale (says Node 2 not locked) versus CURRENT_STATUS.md (Node 2 complete). I'll flag that. Now the Node 1 lock, investigation report, and the Node 3 plan itself.Now let me inspect the actual source repo — schema, existing trip consumers, and freight_identities/companies tables.Good — I'll pin the ref to the exact investigated commit throughout to match the plan's baseline.This is the critical finding — there is **no `companies` table at all**. `freight_identities` only distinguishes DRIVER/COMPANY at the auth-identity level, with no separate company entity to reference. Let me confirm with migration 005 and check for any companies table elsewhere.Good — a `companies` table *does* exist (migration 005), keyed by `auth_id`. This directly matters for the "receiving company" question. Now let's check the events table and event API routes to assess Workstream D compatibility claims.This is a significant finding: `events.driver_id` is `NOT NULL` and independent of `trips.driver_id`, and the `UNIQUE (trip_id, event_type)` constraint means each trip only ever gets one arrival/checkin/departure — this matters for Node 5, not Node 3, but it confirms events reference driver_id directly, not derived from trip. Let's check the API routes and app pages.This confirms a critical, roadmap-relevant gap: **`trip_id` from the client is trusted with no ownership/assignment check against `trips.driver_id`.** Any authenticated driver can post an arrival event for *any* trip_id. Let me check the driver-facing page and `drivers` table for completeness.Confirms: `.eq('driver_id', driverId).eq('status', 'active')` — nulling `driver_id` and adding `'draft'/'published'` statuses is compatible here, since this query already filters on both. Good — supports the plan's compatibility claim. I have enough to write the review.Good, this is enough evidence. Here's the independent Node 3 review.

---

# NODE 3 PLAN REVIEW — Independent Findings

## 1. Trip schema evolution & migration safety

**CRITICAL — Plan under-specifies `status` migration, real risk with existing rows.**
Current `trips.status default 'active'`, and the Historical Core MVP driver dashboard (`page.tsx`) queries `.eq('status','active')`. The plan says "add 'draft', 'published'" but never states what happens to *existing* `'active'` rows or whether `'active'` itself is kept as a legal state (it must be — CLAIMED/IN_PROGRESS trips created under the old model still say `'active'` and are being displayed by the exact query above). The migration plan (Workstream A) must explicitly enumerate: full allowed status set post-migration, and confirm `'active'` is preserved as a legal value (not silently renamed/mapped), or existing "driver's active trip" views break. This must be resolved *before* implementation, not discovered during it.

**IMPORTANT — No explicit CHECK constraint requirement for `status`.**
Currently `status` is unconstrained free text. Node 3 plan doesn't require adding a CHECK/enum constraint even though it's introducing a formal state machine (DRAFT → PUBLISHED per Node 1 lock). Without a constraint, nothing stops a bad write from putting a trip in an invalid state — this is a real IDOR/data-integrity gap, not stylistic. Recommend adding a CHECK constraint scoped to Node 3's legal values plus the pre-existing `'active'`.

**NO CHANGE NEEDED — `driver_id` nullability approach.**
Dropping `NOT NULL` on `trips.driver_id` is correctly identified and is safe given confirmed FK-only usage; no CASCADE issues found in `events` (which references `driver_id` independently, not through `trips.driver_id`).

## 2. Driver assignment nullable — compatibility

**NO CHANGE NEEDED**, verified directly against source. `page.tsx`'s driver dashboard query already filters `.eq('driver_id', driverId).eq('status','active')`, so null-driver / draft / published trips are naturally excluded from a driver's own view. The investigation report's claim here is accurate.

## 3. Sending/receiving Company relationships

**CRITICAL — Plan treats `company_id` as merely "an implementation candidate," but a `companies` table already concretely exists** (`005_v2_onboarding_evidence.sql`), keyed `companies.auth_id → auth.users.id`. The plan and investigation report both under-state this — the investigation report says "Gap: Need `company_id` referencing a company" without noting the target table already exists and is keyed by `auth_id`, not by `freight_identities.id`. This is a real design decision Antigravity needs pinned down explicitly: **trips.company_id must reference `companies(id)`**, not `freight_identities(id)` or `auth.users(id)`. Left ambiguous, this is exactly the kind of "silently invented" data model the plan says to avoid (Section 5.3) — yet the plan itself doesn't resolve it, it just restates the ambiguity. Fix: state explicitly in the plan (or the implementation prompt) that both `company_id` (creator) and `receiving_company_id` should be `uuid references companies(id)`.

**IMPORTANT — Receiving company existence assumption unverified.**
Node 1 lock allows "Sending Company may equal Receiving Company," implying the receiver is also a first-class `companies` row the creating company selects from a set. But nothing in the current source shows companies can browse/select each other (no company listing endpoint, no visibility rule beyond each company reading its own row via RLS: `"Companies can view their own profile" ... USING (auth_id = auth.uid())`). For a Create Trip UI to let a company "select/enter receiving Company" (plan §6, Workstream C), a company must be able to look up *other* companies by name — which the current RLS policy explicitly blocks (SELECT policy is restricted to your own row only). **This is a real, unaddressed blocker**: either (a) add a policy/endpoint allowing limited company lookup (e.g., search by name, minimal fields), or (b) Node 3 uses free-text receiver entry for the hackathon and the FK relationship is deferred — but the plan must pick one, not leave it for Antigravity to improvise mid-implementation, which is exactly the failure mode Node 1 explicitly discourages ("Do not silently change a locked model").

## 4. Draft → Published lifecycle

**NO CHANGE NEEDED** on the state names themselves (DRAFT → PUBLISHED matches Node 1 lock §4). Correctly scoped to stop at PUBLISHED/AVAILABLE, deferring CLAIMED+.

## 5. Company authorization / IDOR

**IMPORTANT — Plan's Workstream B language is sound, but existing sibling code (events API) already demonstrates a class of vulnerability the plan should explicitly guard against by name.** `POST /api/events/arrival` trusts a client-supplied `trip_id` with zero check that the trip belongs to the authenticated driver — any authenticated driver can insert an arrival event onto someone else's trip today. That's a Node 6 issue, not Node 3's to fix, but it's directly relevant precedent: the Node 3 plan's publish/create endpoints must not repeat this pattern (client-supplied trip ownership without server verification). The plan text already says the right things ("Verify the acting Company owns/is authorized for the trip") — flagging this as a place where Antigravity has a known-bad local pattern to *not* copy. Worth an explicit one-line callout in the implementation prompt referencing this exact class of bug.

**NO CHANGE NEEDED** — the plan correctly forbids trusting client-supplied company/owner IDs.

## 6. Compatibility with existing driver/event/timeline code

**NO CHANGE NEEDED**, per source verification in §2 above.

## 7. Missing Node 3 requirements

**MINOR — "Distance" and "Duration" fields have no defined source.** Node 3 roadmap lists Distance/Duration as required fields, but there's no geocoding/routing integration anywhere in the current source or plan. The plan should clarify whether these are manually entered by the company (consistent with "manual offer" allowance) or computed — otherwise Antigravity may guess and add an unplanned dependency (e.g., a maps API). Given hackathon scope and the "manual offer" precedent, manual entry is the obviously correct choice — the plan should say so explicitly rather than leaving it implicit.

**MINOR — No mention of what "eligible drivers" means for making a trip "available."** Node 1 says "before claim: ANY AUTHENTICATED DRIVER" — so eligibility is trivial for Node 3 (all verified drivers). Fine as-is, but plan's acceptance criteria say "Published trip becomes available according to eligibility rules" without stating those rules are simply "no filter yet." Should be made explicit so Ayush's manual verification isn't checking for eligibility logic that doesn't exist yet.

## 8. Scope creep into Node 4+

**NO CHANGE NEEDED.** Section 9 (Explicit Non-Goals) correctly excludes marketplace/claim/atomic-acceptance work. Nothing in the plan's workstreams actually implements claim logic.

## 9. Unsupported assumptions

Covered above — the `company_id`/`receiving_company_id` FK target and the receiving-company lookup/visibility mechanism are the two live unsupported assumptions that need resolving before implementation, not during it.

## 10. Acceptance criteria sufficiency

**IMPORTANT — Acceptance criteria don't test the cross-company IDOR case explicitly enough.** "Unauthorized company/user actions are rejected" (ROADMAP.md) and "Unauthorized Company cannot publish another Company's trip" (plan §11) are present, but there's no criterion checking that Company A cannot even *view/read* Company B's draft trip (read-path IDOR, not just write-path). Given the RLS pattern already in place elsewhere (`companies` table restricts SELECT to own row), the trips table needs an equivalent read-boundary explicitly tested — recommend adding an acceptance-criteria line for "Company A cannot read Company B's trip via direct ID."

---

# VERDICT: APPROVE WITH CHANGES

## Required changes before implementation begins

1. **Pin the target companies model explicitly**: `trips.company_id uuid references companies(id)` and `trips.receiving_company_id uuid references companies(id)` — state this in the plan/prompt rather than leaving it as an "implementation candidate."
2. **Resolve receiving-company selection mechanism**: either add a scoped company-lookup capability (name search, minimal-field policy) or explicitly decide Node 3 uses free-text receiver entry for the hackathon with the FK deferred. Don't leave this for Antigravity to improvise.
3. **Explicitly enumerate the full legal `status` value set** post-migration, confirming `'active'` remains valid (not renamed) so the existing driver dashboard query keeps working, and add a CHECK constraint scoped to the enumerated values.
4. **Clarify Distance/Duration as manually entered** by the company, consistent with manual-offer precedent, to prevent scope creep into routing/geocoding integration.
5. **Add an explicit read-path IDOR acceptance criterion**: Company A cannot fetch/view Company B's trip by ID, not just "cannot publish it."
6. **Add a one-line caution in the implementation prompt** against repeating the existing `events` API's trust-client-supplied-ID pattern (not a Node 3 fix, just a precedent to avoid copying).

None of these are large — they're clarifications and one added constraint — but each is a place where the plan currently defers a real decision to implementation time, which is the exact failure mode Node 1/Node 3 process is designed to prevent. Once resolved, this plan is otherwise sound and ready for an implementation prompt.
