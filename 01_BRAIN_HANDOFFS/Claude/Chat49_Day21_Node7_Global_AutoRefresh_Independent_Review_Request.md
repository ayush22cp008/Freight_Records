# Chat49 — Day 21 — Node 7
## Claude Independent Review Request: Global Auto-Refresh

**Review type:** Independent peer-brain / red-team architecture review
**Status:** REQUEST FOR REVIEW — do not implement

## Context

Freight is currently in Node 7 — AI + Final Integration + Demo. Driver, Company, and Reviewer portal baselines are locked.

The user requirement is broader than dashboard refresh:

> During normal use, the entire Freight website should update relevant visible application state automatically so the user does not need to manually refresh the browser anywhere.

Two investigation records currently exist:

- `05_DEBUGGING/investigations/Chat49_Day21_Node7_Report_CrossPortal_AutoRefresh.md`
- `05_DEBUGGING/investigations/Chat49_Day21_Node7_CrossPortal_Global_AutoRefresh_Investigation_Report.md`

The current Antigravity investigation recommends a lightweight `GlobalAutoRefresh` Client Component mounted in the authenticated layout and calling `useRouter().refresh()` every 30 seconds. It claims this can preserve the existing Server Component architecture and avoid API/RLS changes, while acknowledging database-load and stale-window risks.

## Your Task

Perform an independent, skeptical review of the proposed architecture and the investigation evidence. Do not assume the existing recommendation is correct.

Inspect the current Freight source repository and relevant Records before reaching a conclusion.

## Questions Claude Must Answer

1. Is the claimed global coverage of a layout-level `router.refresh()` technically correct for the actual application architecture?
2. What pages/routes/data surfaces would actually refresh, and which could remain stale?
3. Is a single 30-second global poller appropriate, or should refresh behavior differ by route/state/surface?
4. What important risks or edge cases are missing from the current investigation?
5. Could repeated `router.refresh()` calls negatively affect existing UI behavior, forms, dialogs, filters, navigation, local state, loading states, or user interactions?
6. Could it cause excessive Supabase/database/API load at realistic concurrent-user/tab counts?
7. What happens with multiple tabs, hidden/background tabs, sleeping devices, slow networks, offline/online transitions, and overlapping refresh requests?
8. Could concurrent refreshes expose stale or out-of-order state, or create race conditions in the UI?
9. Are there pages whose data should update immediately/eventually but cannot be safely handled by periodic `router.refresh()`?
10. Is Supabase Realtime actually unnecessary, or are there specific workflows where realtime would materially improve correctness/UX?
11. Does the proposed mechanism preserve the protected boundaries: API contracts, RLS/security, authentication/authorization, business rules, persistence, lifecycle semantics, evidence semantics, and locked portal behavior?
12. Could introducing the global client wrapper itself alter the behavior of the locked portals enough to require governance reopening?
13. What testing/evidence would be required to establish that the change does not harm the existing system?
14. What alternative architecture, if any, is safer or more scalable?

## Required Review Method

Separate clearly:

- VERIFIED — directly supported by source inspection/evidence
- INFERRED — reasoned conclusion not directly proven
- UNKNOWN — requires additional evidence

Do not propose implementation merely because the existing report recommends it.

## Required Output

Produce a concise but technically rigorous review record containing:

- agreement/disagreement with the current investigation;
- missed risks or incorrect assumptions;
- recommended architecture, if different;
- conditions/safeguards required for a safe implementation;
- minimum verification/test matrix needed before approval;
- and an explicit recommendation: APPROVE / REVISE / REJECT current approach.

No source-code changes are authorized by this review request.

## Governance

Do not create an implementation prompt from this review. The review is advisory only. Final decision and approval remain with Ayush.

**Project:** Freight — AI Builders Hackathon
**Node:** Node 7
**Day:** Day 21
**Chat:** Chat49
