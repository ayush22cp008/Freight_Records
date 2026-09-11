# Chat50 — Day21 — Node7
## Same-Company Sender = Receiver Governance Decision

**Decision owner:** ChatGPT (architecture/reasoning)

**Final authority:** Ayush

**Execution agent:** Antigravity

**Decision status:** APPROVED BY AYUSH

**Implementation authorization:** NOT GRANTED BY THIS RECORD

**Required architecture action:** REOPEN / SUPERSEDE the previously locked Company same-company behavior before implementation.

---

## 1. Decision Trigger

The Chat50 governance reinvestigation established that allowing a Company to select itself as both Sending Company and Receiving Company is not an accidental UI-only defect. It is an intentionally supported behavior in the current locked Company architecture and backend implementation.

Ayush has now explicitly chosen to change that product rule.

Authoritative investigation:

`05_DEBUGGING/investigations/Chat50_Day21_Node7_SameCompany_Sender_Receiver_Governance_Reinvestigation_Report.md`

---

## 2. Final Product Decision

### DECIDED

**A Company must NOT be allowed to select itself as the Receiving Company when it is the Sending Company for a NEW Trip.**

Canonical rule:

```text
Authenticated Sending Company = S
Selected Receiving Company = R

S == R
→ REJECT Trip creation

S != R
→ continue normal Receiver selection / agreement workflow
```

This is a product/business-rule change from the previously accepted same-company bypass behavior.

---

## 3. Scope of the Decision

This decision applies to **new Trip creation**.

The prohibition is a system invariant, not merely a visual UI preference.

Therefore the rule must ultimately be enforced across every authoritative Trip-creation path capable of assigning `receiving_company_id`.

A client-side-only restriction is insufficient.

---

## 4. Required System Invariant

For every newly created Trip requiring a Sending Company and Receiving Company:

```text
Trip.company_id != Trip.receiving_company_id
```

When this invariant is violated during a new Trip creation attempt:

```text
Trip creation must fail safely.
No valid new Trip may be persisted with the same Company in both roles.
```

The exact API error/status/message is an implementation detail and must be determined from the existing project conventions during preflight.

---

## 5. Company UI Decision

The Company Trip Creation UI must not present the authenticated Sending Company as a valid Receiving Company option for a new Trip.

The preferred UX is:

```text
Receiving Company selector
        ↓
exclude authenticated Sending Company
        ↓
user selects another Company
```

If current source architecture makes the authenticated Company appear in the selector through an existing query, the implementation must correct the actual data/selection path rather than merely hide text after selection.

The exact UI approach remains implementation detail after source inspection.

---

## 6. Server-Side Enforcement Decision

The backend must independently enforce:

```text
authenticated sender Company
        ↓
selected receiving Company
        ↓
if equal
        ↓
reject creation
```

The server must not trust a client assertion that the selected receiver is external.

The authoritative comparison must use authenticated identity and the persisted Trip/company relationship data available at creation time.

This prevents API/manual-request bypasses.

---

## 7. Receiver Request / Handshake Impact

The previous architecture contained a same-company exception:

```text
Sender == Receiver
→ no external Receiver handshake
→ server-derived ACCEPTED bypass
```

That exception is no longer valid for NEW Trips.

For new Trips under the revised rule:

```text
Sender != Receiver
→ Receiver Delivery Request workflow applies
```

Because same-company Trips are prohibited at creation, the new implementation must not create a same-company Receiver Request as a normal supported creation path.

---

## 8. Legacy / Existing Trips

This architecture decision does **not** authorize destructive migration of existing data.

Existing same-company Trips may already exist because the previous architecture explicitly permitted them.

Therefore:

```text
Existing same-company Trips
→ preserve until separately assessed
```

No existing Trip should be deleted, rewritten, reassigned, or retroactively given a fabricated Receiver decision merely because the product rule has changed.

Before implementation is considered complete, the implementation handoff must explicitly verify the treatment of existing same-company records, including operational and historical Trips.

**Legacy treatment:** OPEN IMPLEMENTATION-SAFETY QUESTION, not authorization for destructive change.

---

## 9. Locked Blueprint Reopen

The current locked Company blueprint explicitly contains the previous same-company behavior.

Therefore the change cannot be represented honestly by silently editing unrelated implementation files while leaving the locked blueprint unchanged.

Required governance sequence:

```text
Current locked same-company rule
        ↓
Chat50 approved product decision
        ↓
Formal architecture reopen / superseding decision
        ↓
Updated authoritative Company architecture
        ↓
Implementation plan / prompt
        ↓
Antigravity execution
        ↓
Testing + Ayush manual verification
        ↓
Re-lock / approval update if required
```

The historical locked blueprint remains preserved as a historical record. The new decision supersedes its same-company rule for new Trip creation once the architecture reopen is formally applied.

---

## 10. Protected Existing Behaviors

This decision changes only the same-company Sender/Receiver product rule.

It does **not** authorize unrelated changes to:

- Driver Portal
- Reviewer Portal
- authentication
- authorization model
- Driver claim transaction
- evidence workflow
- delivery lifecycle
- Receiver operational Check-in / Completion
- History semantics
- Public Share
- AI behavior
- Cross-Portal auto-refresh
- any unrelated Company UX

Any additional behavior change requires separate evidence and governance.

---

## 11. Required Implementation Investigation Before Coding

Before an implementation prompt is issued, Antigravity must verify the real source paths for:

1. Company Trip Creation UI.
2. Receiving Company lookup/query.
3. Authenticated Company identity resolution.
4. `POST /api/trips/create` or its actual current equivalent.
5. Any alternate server action/API capable of creating Trips.
6. Same-company handling in Receiver Request creation.
7. Any legacy/migration path that could create or modify Sender/Receiver relationships.

The preflight must also identify whether any non-UI creation path exists.

No coding should begin until this preflight confirms the actual implementation boundary.

---

## 12. Required Test Coverage

The future implementation must prove at minimum:

```text
Case A:
Sender A → Receiver B
→ ALLOWED

Case B:
Sender A → Receiver A
→ REJECTED

Case C:
UI attempts to select Sender A as Receiver
→ option unavailable / prevented

Case D:
Direct API request attempts Sender A → Receiver A
→ server rejects

Case E:
Existing legacy same-company Trip
→ behavior preserved unless separately approved for migration change
```

The exact expected status codes/messages and test harness must follow the actual project conventions.

---

## 13. Implementation Boundary

This record is an **architecture/governance decision**, not an implementation prompt.

Therefore:

```text
Implementation authorization from this record
→ NO
```

A separate implementation plan/prompt must be created after the architecture reopen is completed and the source preflight identifies the exact files/APIs involved.

Antigravity must not infer implementation scope directly from this decision and begin coding without the explicit implementation handoff.

---

## 14. Final Decision Statement

### APPROVED

For the Freight product going forward:

> **The Sending Company cannot also be selected as the Receiving Company when creating a new Trip.**

This replaces the previous same-company bypass behavior for new Trip creation.

The change is intentionally governed as a product-rule change because the previous behavior was already part of the accepted Company architecture.

**Status:** APPROVED PRODUCT DECISION / ARCHITECTURE REOPEN REQUIRED / IMPLEMENTATION NOT YET AUTHORIZED
