# Chat30 — Node 7 Phase 1a Public Evidence Sharing Decision Checkpoint

**Project:** Freight — AI Builders Hackathon  
**Node:** Node 7 — AI + Final Integration + Demo  
**Phase:** Phase 1a — Baseline AI + Shareable Evidence  
**Date:** 2026-09-03 — Day 14 / Chat30  
**Reasoning brain:** ChatGPT  
**Status:** DECISION CHECKPOINT / APPROVED BY AYUSH

## Checkpoint Purpose

This checkpoint records the completed one-by-one architecture and product/security decision discussion for the Node 7 Phase 1a public shareable read-only evidence link.

No source-code implementation was performed as part of this checkpoint. No prior Node was reopened. The purpose is to preserve the decisions before formal reconciliation against the existing Freight source and Node 6 security baseline.

## Approved Node 7 Phasing Context

Node 7 was previously approved in Chat29 with the following execution sequence:

```text
Phase 1a — Baseline AI + Shareable Evidence
        ↓
Phase 1b — Full 3-Portal UI/UX Redesign
        ↓
Phase 3 — Conditional Add-On Features
        ↓
Final Step — E2E + Bugfix + Demo + Presentation
```

Phase 1a includes:

- AI evidence-grounded summary
- Timeline integration
- Public shareable read-only evidence link

The official Node 7 acceptance criteria remain unchanged.

## Existing-System Investigation Basis

Before this decision checkpoint, the current source and Records were inspected.

Verified existing capabilities include:

- `/api/summary` exists and authenticates the requesting user, resolves the authenticated driver, loads a specific trip's events, checks required evidence, and produces an evidence-grounded factual AI summary.
- The Timeline page supports an exact `tripId` and passes the exact trip ID into `AIEvidenceSummary`.
- The existing Timeline exposes event type, timestamps, GPS, GPS accuracy, and photo evidence to authenticated users.
- No existing public share/evidence-share route was found.
- No existing migration establishing public evidence sharing was found.
- Node 6 security verification established server-side authorization, IDOR protection, role/relationship boundaries, evidence immutability, and rate-limiting verification.

Therefore:

```text
Existing AI summary              → VERIFIED / EXISTING
Existing Timeline integration    → VERIFIED / EXISTING
Public evidence sharing          → VERIFIED GAP / NEW NODE 7 WORK
```

## Public Evidence Sharing Decisions

### Public content and privacy

1. **Public content:** Trip verification summary + sanitized evidence timeline.
2. **Raw photos:** Hidden from the public view.
3. **Precise GPS:** Hidden from the public view.
4. **Public location:** City/area-level only.
5. **AI summary:** Public.
6. **Driver identity:** Hidden.
7. **Company name:** Public.
8. **Company logo:** Public when available from the existing company profile; otherwise company name only.
9. **Public-safe trip/reference number:** Public.
10. **Internal company/customer details:** Hidden.
11. **Vehicle information:** Use existing safe vehicle data if available; otherwise vehicle type or omit it. Do not create a new vehicle-data model solely for public sharing.
12. **Delivery date:** Public.
13. **Pickup and destination:** Public at city/area level.
14. **Public route map:** Not included in Phase 1a.
15. **Event timestamps:** Public.
16. **Event types:** Public.
17. **GPS accuracy:** Hidden.
18. **Street/full addresses:** Hidden; city/area only.
19. **Trip status:** Show factual `Completed` status.
20. **Share/generated timestamp:** Not shown.
21. **Read-only indicator:** Public page clearly identifies itself as a public read-only verification view.
22. **Public photos:** None.
23. **Public downloads:** None; browser view only.
24. **Public copy/share control:** No management/share control on the public page.

### Access and authorization

25. **Public access:** Anyone possessing the non-guessable share link can view the restricted public representation.
26. **Public authentication:** No Freight login is required for the public verification page.
27. **Share scope:** Exactly one share link/token represents exactly one completed trip.
28. **Link creation:** Company only.
29. **Trip eligibility:** Completed trips only.
30. **Required evidence gate:** Required evidence must be present before initial public-link creation.
31. **Pre-completion creation:** Backend must strictly reject share creation before completion; UI should also hide the Share control before completion.
32. **Company management:** Company can create, copy, and revoke from the Company portal.
33. **Reviewer role:** Reviewer may view share-link status/link but cannot create or revoke.
34. **Driver role:** Driver may see share-link status but cannot create or revoke.
35. **Public role:** Public recipient can only view the restricted verification page.
36. **Role boundary:** Company controls publication; Reviewer/Driver consume status; public recipient consumes the verification artifact.

### Token and URL security

37. **URL design:** Public URL contains only an opaque high-entropy random token.
38. **Public reference vs authorization:** Public reference identifies the delivery; share token authorizes access. They must not be the same credential.
39. **Token persistence:** Store only a secure cryptographic hash of the share token in the database; do not persist the raw bearer token.
40. **Token lifecycle:** Every newly created share uses a fresh independent token.
41. **Revoked token reuse:** A revoked token remains permanently invalid and must never be reactivated.
42. **Active links per trip:** At most one active public share token exists for a completed trip.
43. **Replacement behavior:** Creating a new link automatically revokes the previous active link and activates the new token.
44. **Revocation:** Company can revoke an active link.
45. **Revocation confirmation:** Normal confirmation is required before revocation.
46. **Revoked/invalid public view:** Show the same generic unavailable response for invalid, revoked, nonexistent, or otherwise unusable tokens.
47. **Anti-enumeration:** Public behavior must not distinguish token states in a way that leaks whether a token/trip existed.
48. **Rate limiting:** Public verification endpoint must be rate-limited. Numeric thresholds are not defined here and must align with the established Node 6 rate-limiting architecture.
49. **Search indexing:** Public verification pages must not be indexed by search engines.
50. **Caching:** Apply appropriate no-cache/private caching controls to reduce stale public content after revocation. Exact cache-control policy must be aligned with deployment architecture.

### Evidence, AI, and consistency behavior

51. **AI section:** AI summary and evidence timeline are separate public sections.
52. **AI role:** AI is an evidence-grounded enhancement; recorded evidence remains primary.
53. **AI disclaimer:** Include a concise factual disclaimer that recorded evidence is primary and the AI summary is generated from available evidence.
54. **AI failure:** If AI generation is unavailable, keep the public timeline available and show a generic AI-unavailable message.
55. **AI error details:** Do not expose provider/API/model error details publicly.
56. **AI provider/model:** Do not expose the AI provider or model name publicly.
57. **AI consistency:** Public AI summary must correspond to the current permitted public evidence state.
58. **Live view:** Public verification is a live read-only representation of the current approved public data, not a separate immutable snapshot.
59. **Evidence changes:** If evidence later becomes incomplete, keep the link active unless separately revoked and show the current evidence state.
60. **Evidence completeness:** Show explicit evidence completeness based on existing required-evidence rules; do not invent a numerical score.
61. **Evidence checklist:** Show a simple human-readable checklist of required evidence categories present.
62. **Completeness explanation:** Show the checklist, not internal validation algorithms or technical decision logic.
63. **Current state:** Public page shows current permitted state rather than internal mutation/audit history.
64. **Required evidence after changes:** Public state may become `Incomplete` if current required evidence is no longer present; do not silently present an outdated complete state.

### Public UX and data boundary

65. **Final public architecture:** Public verification must be implemented through a dedicated public route/API boundary, separate from existing authenticated APIs.

The public representation should remain limited to the explicitly approved contract:

```text
Company identity
→ Public trip reference
→ Delivery date
→ Completed status
→ Pickup/destination city/area
→ Evidence completeness
→ Required-evidence checklist
→ AI evidence-grounded summary
→ Event timeline with event type + timestamp + city/area
→ Read-only indicator
```

The following remain private:

```text
Driver identity
Raw photos
Exact GPS
GPS accuracy
Street addresses
Internal IDs
Internal company/customer details
Internal vehicle identifiers/registration details
AI provider/model details
Security/audit implementation details
```

## Link Lifecycle

```text
Completed trip + required evidence
            ↓
      Company creates link
            ↓
     Fresh random token
            ↓
       Token hash stored
            ↓
       Public link Active
            ↓
     Public read-only access
            ↓
Company creates replacement link
            ↓
   Old token automatically revoked
            ↓
      New token becomes active
```

A company may also explicitly revoke the active link:

```text
Active → Confirm revoke → Revoked permanently
```

A revoked or invalid token always results in the generic unavailable response.

## Public Verification Failure Model

The public verification page must distinguish conceptually between:

```text
AI unavailable        → timeline remains available
Evidence incomplete   → current evidence state is shown
Token invalid/revoked → generic unavailable page
```

AI failure must not make the entire evidence verification page unavailable.

## Security Boundary

The core architecture decision is:

```text
Existing authenticated Freight APIs
        → remain authenticated/protected

Dedicated public verification route/API
        → validates opaque share token
        → checks active/revoked state
        → enforces one-trip scope
        → enforces completed-trip eligibility
        → returns only approved public fields
        → applies rate limiting
        → avoids token-state enumeration
        → applies appropriate cache controls
        → records minimal internal access audit event
```

The public endpoint must not weaken or convert existing authenticated endpoints into mixed public/authenticated APIs.

## Internal Audit Decisions

Public sharing actions and access should be auditable internally:

- Share link created → audit internally.
- Public share link accessed → audit minimally internally.
- Share link revoked → audit internally.
- Replacement share created → audit as creation plus prior-link revocation as applicable.

Audit records are private and must align with the existing Node 6 audit/security architecture. Exact audit fields are not defined in this checkpoint.

## Scope Protection

The following are intentionally outside the Phase 1a baseline unless later approved as conditional Phase 3 work:

- Public raw-photo viewing
- Public exact GPS
- Public street addresses
- Public route map
- Public PDF/export/download
- Automatic link expiration
- Configurable public-sharing policies
- Public AI model/provider details
- Numerical evidence-completeness scoring
- Immutable public snapshots
- New vehicle-data model solely for public sharing
- Expanded public audit/visitor analytics

## Verification / Unknowns Before Implementation

This checkpoint records decisions, not implementation approval. Before implementation, the decisions must be reconciled against the current source and Records.

Items requiring explicit source/Records verification include:

- Existing company profile/logo fields and their public-safe serving mechanism.
- Existing vehicle fields and whether a safe vehicle type/reference exists.
- Existing required-evidence logic and exact evidence categories used by the current AI summary flow.
- Existing company-to-trip authorization path for the Company portal.
- Existing Node 6 rate-limiting architecture and appropriate public-endpoint threshold.
- Existing audit/logging architecture and minimal fields appropriate for share create/access/revoke.
- Deployment behavior for public routes and cache-control headers.
- Existing application routing conventions for a dedicated public route.
- Safe handling of any existing storage URLs so raw photo evidence cannot become publicly reachable through the new route.

These items must not be treated as verified until checked.

## Decision Outcome

```text
Node 7 Phase 1a public-share architecture discussion → COMPLETE
Questions/decisions settled                         → 65
Public evidence sharing contract                    → DECIDED
Implementation                                      → NOT STARTED
Source/Records reconciliation                       → NEXT
Ayush approval of this checkpoint                   → APPROVED
```

## Governance / Handoff

ChatGPT remains the architecture/reasoning/investigation brain for this checkpoint. Antigravity remains the implementation/execution agent. GitHub Records remains the source-of-truth bridge.

This checkpoint does not authorize implementation by itself. The next active reasoning step is to reconcile this decision set with the existing Freight source and Node 6 security baseline, identify VERIFIED/GAP/UNKNOWN items, and then prepare the formal implementation/investigation handoff through the Records repository according to the project workflow.

No prior Node is reopened by this checkpoint.
