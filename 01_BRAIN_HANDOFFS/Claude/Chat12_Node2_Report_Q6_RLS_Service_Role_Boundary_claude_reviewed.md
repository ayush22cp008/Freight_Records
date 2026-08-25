Checked the six corrections against the current text:

**1. RLS default-deny behavior** — Present. §4 says "protected Freight business tables have RLS enabled unless a documented architecture exception," and §12 test 11 explicitly requires fail-closed behavior for missing/restrictive policies. Good.

**2. FORCE ROW LEVEL SECURITY / table-owner bypass** — **Missing.** Nowhere in the doc does it mention `FORCE ROW LEVEL SECURITY`. This matters because Supabase/Postgres RLS policies do *not* apply to the table owner by default — only to non-owner roles — unless `FORCE ROW LEVEL SECURITY` is set. If any app-facing role ends up owning the table (or inherits owner privileges), RLS could be silently bypassed even with correct policies. §4 and §10 talk about "RLS enabled" but never address the owner-bypass nuance. This correction did not make it in.

**3. service_role compromise and rotation** — **Missing.** §3 (Threat Model) lists "leaked service-role credentials" as a threat but there's no corresponding mitigation anywhere — no mention of rotation procedure, blast-radius limiting, or response-if-compromised. The threat is named but never answered.

**4. Mandatory audit logging for security-sensitive privileged mutations** — **Weakened/not incorporated as "mandatory."** §10 point 7 says "Privileged operations *should be* identifiable in logs/audit records *where appropriate*" — this is hedged, optional language, not a mandatory requirement. If the correction was to make audit logging mandatory specifically for security-sensitive mutations (verification_status, trusted_role, admin approvals), that stronger requirement isn't reflected.

**5. Strict service_role exception wording** — Present and reasonably strict. §5, §10.3, §10.8 use "reserved for," "genuinely require," "not merely to avoid fixing an RLS policy." This one looks incorporated.

**6. Separate RLS vs Node 1 acceptance tests** — Partially present. §7 does separate the concepts narratively, and §12 test 10 says "RLS and Node 1 authorization both apply where required; passing one does not bypass the other" — but this is a single combined test, not separate RLS-specific and Node-1-specific acceptance tests. If the correction asked for two distinct test entries (one proving RLS enforcement independent of Node 1, one proving Node 1 enforcement independent of RLS), that split isn't done — it's still merged into one bullet.

---

**Verdict: CHANGES REQUIRED**

**Remaining corrections needed:**
1. Add explicit `FORCE ROW LEVEL SECURITY` requirement to §4/§10 — state that table ownership must not grant implicit RLS bypass, and that FORCE RLS (or equivalent non-owner app role) is mandatory for protected tables.
2. Add a service_role compromise/rotation subsection (likely under §5 or a new §5.x) — minimum: key rotation procedure/cadence, and immediate-response steps if a leak is suspected. Currently only listed as a threat, never mitigated.
3. Upgrade §10 point 7 from "should be identifiable... where appropriate" to a mandatory requirement specifically for security-sensitive mutations (verification_status, trusted_role, admin approval changes) — every such mutation must produce an audit record, not just "where appropriate."
4. Split §12 test 10 into two separate acceptance tests: one proving RLS blocks cross-user access independent of Node 1 authorization state, and one proving Node 1 authorization blocks disallowed operations even when RLS would permit the row access.

**Other observations (not part of the 6, flagging since asked for remaining issues):**
- §8 mentions "Auth-trigger-based identity creation may use database-level privileges" but doesn't specify whether that trigger runs as `SECURITY DEFINER` and if so, what constrains its scope — worth a verification item in §13 if not already implied.
- §13 item 5 ("source locations where service_role is currently imported/used") is good but could explicitly ask whether `src/lib/supabase-server.ts` is imported anywhere outside verified privileged routes — ties directly to the IDOR risk in §9.

**Final Q6 recommendation:** The core policy (Strict RLS + Privileged Server Boundary) is sound and should stand. Do not reopen the architecture decision — only the four gaps above need closing before this can move to independent review/lock. Once FORCE RLS, service_role rotation, mandatory audit logging, and split acceptance tests are added, this is lock-ready.
