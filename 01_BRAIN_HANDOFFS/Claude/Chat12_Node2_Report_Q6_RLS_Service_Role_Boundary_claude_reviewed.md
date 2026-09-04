Independent review, Q6 only. No implementation done.

## Verdict: **CHANGES REQUIRED** (minor — content is close to solid, but has gaps that matter for an MVP with real user data)

## Problems / Concerns

**1. "Fail closed" claim isn't actually enforced anywhere concrete (§12, item 11)**
Test #11 says a missing/restrictive RLS policy should "fail closed," but nothing in §4 or §10 states the operational rule that makes this true by construction: *RLS enabled with no policy = deny all by default* is Postgres/Supabase's actual behavior, but the report never states this explicitly as the reason it's safe. Without saying it, someone reading only this doc could assume "RLS enabled" alone is sufficient, without also confirming policies exist per operation — leading to a table that's RLS-enabled but has zero policies, silently blocking legitimate access (or, if RLS is force-enabled incorrectly, silently allowing service-role paths through unintended grants). Needs one explicit sentence.

**2. No mention of `FORCE ROW LEVEL SECURITY` for table owners**
In Supabase/Postgres, RLS policies don't apply to the table owner role by default unless `FORCE ROW LEVEL SECURITY` is set. If any Freight tables are queried by a role that owns the table (common misconfig), RLS can be silently bypassed even without service_role. This is a real, common gotcha and the report is silent on it. Should be added to §4 or the acceptance tests.

**3. Service-role key rotation / compromise response is missing**
§10 point 5 says the key must never be exposed — good — but there's no policy for what happens *if* it leaks (rotation procedure, blast radius). For a document meant to be the final word before Q6 LOCK, this is a reasonable gap to flag even if deferred, so it doesn't get silently forgotten.

**4. "Auditability... where appropriate" (§10.7) is too soft**
This is the one place the document hedges. Given that Q6's own threat model (§3) explicitly includes "users modifying verification/trusted-role state" and "service_role misuse," audit logging for privileged verification/admin mutations shouldn't be optional/"where appropriate" — it should be a hard requirement for any operation that touches `verification_status` / `trusted_role`, specifically. As written, an implementer could reasonably skip logging entirely and still claim compliance.

**5. Access matrix (§6) "Not preferred" is ambiguous**
For rows like "Read own permitted data → Privileged Server: Not preferred," it's unclear if this means "technically allowed but discouraged" or "forbidden except via approved exception." Given §10.8 ("no shortcut"), this should just say **DISALLOWED except via approved exception**, matching the rest of the document's stricter language. As-is it's the one soft spot an implementer could exploit to justify convenience use of service_role.

**6. Node 1 interaction section is correct but has no failure-mode test**
§7 correctly separates RLS (row access) from Node 1 (business authorization), and states passing one doesn't bypass the other. But §12 has no acceptance test that specifically verifies *Node 1 rejects an action RLS would have allowed* (e.g., Driver A reads their own row via RLS, but Node 1 correctly blocks the operation due to trip state). Test #10 gestures at this but is too generic to be checkable. Should be split into two concrete tests.

## Exact Corrections Needed

1. **§4** — add: "RLS-enabled tables with no explicit policy default-deny all access for non-owner roles; this is the mechanism that satisfies 'fail closed' — not an assumption, an explicit deny-by-default guarantee to verify per table."
2. **§4 or §13** — add unknown/verification item: "Confirm `FORCE ROW LEVEL SECURITY` status for each protected table, to rule out table-owner-role bypass."
3. **§10** — add point 9: "If service_role key exposure is suspected, rotate immediately via Supabase dashboard and audit recent privileged-operation logs for the exposure window." (policy statement, not implementation)
4. **§10.7** — reword to: "Privileged operations that mutate `verification_status`, `trusted_role`, or other administrative approval state MUST be logged with actor identity, target resource, and timestamp. Other privileged operations should be logged where appropriate."
5. **§6** — replace "Not preferred" in both cells with "**DISALLOWED except via approved exception**" to match the strictness of §10.8.
6. **§12** — split test #10 into:
   - 10a. "A row that passes RLS (User A reading their own row) can still be correctly rejected by Node 1 authorization due to business-state constraints (e.g., trip state, role)."
   - 10b. "A row blocked by RLS is never reachable even if Node 1 authorization would have allowed the business operation."

## Final Q6 Recommendation

The core architecture — **RLS as default row boundary, service_role as a narrow server-only exception requiring its own authorization check, and RLS/Node 1 treated as complementary, non-substitutable layers** — is correct and matches standard Supabase/Postgres security practice. This should stand as the locked decision.

Before LOCK, fold in corrections #1, #4, #5 as mandatory (they're either safety-relevant defaults or ambiguity that undermines the document's own stated strictness). Corrections #2, #3, #6 can be folded in now or explicitly deferred to Q7/implementation-time as tracked verification items — but if deferred, they must be added to §13's unknowns list so they aren't lost, not simply dropped.

Once #1, #4, and #5 are patched, Q6 is ready to LOCK.
