# Chat39 — Day 15 — Company Portal Final Remaining System-Presence Investigation

## Purpose

This is the **final focused investigation** for the remaining Company Portal frontend/API/data questions that were not established by the previous investigation report.

The objective is evidence collection only. The investigator must inspect the current source-code system and explicitly state when a capability, route, component, data field, or flow **does not exist / is not present in the current system**. Absence must never be silently skipped, assumed to exist, or filled using product expectations.

This investigation must not redesign the Company Portal and must not reopen findings already verified in earlier investigations.

---

## Existing Verified Context — Do Not Duplicate

Treat these as established findings unless direct source evidence contradicts them:

- `companies` represents a generic Company entity; there is no permanent sender/receiver Company type.
- `trips.company_id` and `trips.receiving_company_id` establish trip-specific Company relationships.
- The same Company can be the sender on one trip and receiver on another.
- Public Share management is restricted to the receiving Company for the specific trip.
- Incoming and Completed Company delivery visibility uses `receiving_company_id`.
- Create Trip establishes the creating Company as `company_id` and the selected receiver as `receiving_company_id`.
- Publish requires Company ownership through `company_id`.
- Receiver Check-in flow has already been traced and verified.
- Receiver Completion flow has already been traced and verified, including the identified frontend/API response mismatch.

Do not re-investigate these points unless necessary to connect them to a remaining flow.

---

# Investigation 1 — Exact Public Share System Presence and Exposed Data

Inspect the current source code comprehensively for the Public Share capability.

Determine, with exact evidence:

1. Does Public Share actually exist in the current system?
   - If YES: identify every relevant frontend page/component, API route, database table/query, and public-access route found.
   - If NO: explicitly state **NOT PRESENT IN CURRENT SYSTEM** and provide the evidence/search scope used to reach that conclusion.

2. How is a Public Share created, activated, revoked, and/or viewed?

3. What exact data is returned to the public viewer?
   - Trip identifiers
   - Trip status
   - Sender/Company information
   - Receiving Company information
   - Driver information
   - Pickup/delivery locations
   - Progress/location data
   - Event/timeline data
   - Timestamps
   - Evidence/photo information
   - Any other exposed fields

4. Identify exactly which fields are intentionally excluded from the public response, if this is demonstrable from source code.

5. Determine whether the public view is:
   - read-only;
   - authenticated;
   - unauthenticated/public;
   - token/share-link based;
   - or another mechanism.

6. Determine whether the frontend representation matches the actual API response. Record any mismatch as a concrete finding.

7. Check whether Public Share behavior differs depending on whether the Company is the sender or receiver for that specific trip. Preserve the already-verified receiving-company authorization finding rather than duplicating its investigation.

---

# Investigation 2 — Complete Company Frontend → API → Data/Event Flow Inventory

Build an evidence-based inventory of the **current** Company Portal flows.

At minimum inspect these areas:

1. Company Dashboard / Overview
2. Create Trip
3. Publish Trip
4. Incoming Deliveries
5. Receiver Check-in
6. Receiver Completion
7. Completed Deliveries
8. Public Share management
9. Public Share viewing
10. Company Profile / Account, if present
11. Company Timeline / History, if present
12. Any additional Company-specific page or workflow actually present in the source

For every item, establish:

`Frontend entry → component/page → API route → authorization identity → trip/company relationship → database read/write → event/status effect → frontend-visible result`

### Critical absence rule

For every listed capability, explicitly classify one of:

- **PRESENT — VERIFIED**
- **PARTIALLY PRESENT — VERIFIED**
- **NOT PRESENT IN CURRENT SYSTEM — VERIFIED**
- **PRESENT BUT BROKEN / DEAD PATH — VERIFIED**
- **UNKNOWN — insufficient evidence**

If a requested/expected Company capability is not found in the actual source code, that is a valid investigation result and must be reported plainly.

Do not convert absence into an assumption that the feature is planned or implied.

---

# Investigation 3 — Company History / Timeline Presence

Because the shared navigation has previously been observed to expose a Timeline destination, verify the actual Company-side state.

Determine:

- Does a Company-specific Timeline/History page exist?
- Does `/timeline` support Company users?
- What data source would it use for Company history, if any?
- Does the current route require a Driver record or another role-specific record?
- If Company navigation exposes Timeline but Company cannot actually use it, classify the exact issue as VERIFIED.
- Determine whether an alternative Company history/timeline capability exists elsewhere.

Do not redesign the navigation. Only establish current system truth.

---

# Investigation 4 — Company Profile / Account Presence

Search the current source for Company Profile / Account functionality.

Determine:

- Whether a Company profile/account page exists.
- Its route and frontend component, if present.
- What Company data is displayed.
- Whether the Company can edit any profile data.
- Which API/database source supplies the data.
- Whether the capability is functional, partial, dead, or absent.

If no Company Profile/Account capability exists, explicitly report:

**NOT PRESENT IN CURRENT SYSTEM — VERIFIED**

Do not infer one from generic authentication/account infrastructure.

---

# Investigation 5 — Responsive / Mobile System Presence

Inspect the existing Company frontend implementation for evidence of responsive/mobile behavior.

Determine only what can be established from source:

- responsive layout classes/logic;
- mobile navigation behavior;
- tables/cards/list transformations;
- viewport-specific UI;
- overflow handling;
- forms/actions on smaller screens;
- any explicit mobile-only or desktop-only Company components.

Classify:

- VERIFIED responsive behavior;
- VERIFIED responsive gaps;
- UNKNOWN due to insufficient source evidence.

Do not perform a visual redesign or invent browser-test results. If runtime/manual testing is required but unavailable, mark it UNKNOWN.

---

# Investigation 6 — Shared vs Company-Specific Final Check

Use the existing Shared Components investigation as context and perform only a final consistency check needed for the complete flow inventory.

Identify:

- shared Company-facing components actually used;
- Company-specific components actually used;
- role-based assembly points;
- shared navigation destinations that are not valid for Company users;
- Company flows that exist only through role-specific components.

Do not duplicate the prior shared-component investigation unnecessarily.

---

# Investigation 7 — Final Evidence Matrix

Produce a final evidence matrix covering all remaining Company Portal findings.

Each row must contain:

- Area / capability
- Current-system presence
- Exact source evidence/path
- Frontend entry
- API/data dependency
- Authorization / relationship dependency
- Event/status effect
- Frontend result
- Issue/gap, if any
- Confidence: VERIFIED / INFERRED / UNKNOWN

For absence findings, the evidence column must explain what was searched/inspected and why the absence is considered verified rather than simply omitting the feature.

---

# Investigation 8 — Final Company Existing-Structure Conclusion

End with a strict conclusion answering:

1. What Company Portal capabilities definitely exist today?
2. What capabilities are partially implemented?
3. What capabilities are broken/dead paths?
4. What capabilities are definitely absent from the current system?
5. What remains genuinely UNKNOWN?
6. Which findings materially affect the upcoming Company Mental Model and Interaction Mapping?

This section is still **evidence only**. Do not turn findings into UX decisions or implementation requirements.

---

## Required Investigation Discipline

Follow:

`OBSERVATION → INVESTIGATION → EVIDENCE → ROOT CAUSE (where applicable) → DECISION/IMPLICATION`

Rules:

- Inspect the actual current source code, not assumptions or expected product behavior.
- Use exact file paths, route names, component names, API endpoints, schema/table names, and relevant code references.
- Distinguish frontend presence from backend capability.
- Distinguish route existence from a usable end-to-end flow.
- Distinguish database support from actual Company UI support.
- Explicitly report absence.
- Do not silently fill missing information.
- Use VERIFIED / INFERRED / UNKNOWN for substantive conclusions.
- Do not modify source code.
- Do not create implementation prompts.
- Do not redesign the Company Portal.
- Do not lock the Company Mental Model or Company Blueprint.
- Do not reopen completed Nodes 1–6.
- Do not reopen already-verified Company relationship findings unless required to connect a remaining flow.

## Completion Condition

This investigation is complete only when every scope section above has an explicit evidence-backed result, including explicit **NOT PRESENT IN CURRENT SYSTEM** results wherever applicable.

If a capability is absent, say so clearly. If evidence is insufficient, say UNKNOWN. Do not leave a scope item silently unreported.

The resulting report should be sufficient to close the Existing Company Frontend Structure investigation and provide the factual foundation for the next Company Mental Model step.