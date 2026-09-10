# Reviewer + Driver + Company Truth Audit — Evidence Questionnaire Follow-up Report

**Status:** INVESTIGATION ONLY
**Investigator:** Antigravity
**Date:** 2026-09-10

## 1. Environment Questions

### Q1. What exact application/runtime are you investigating?
**Repository:** Local file system (`C:\Users\ayush\Desktop\Freight_hackathon\freight`)
**Branch:** main
**Commit SHA:** `8fa10cc` (Freight_Records repository)
**Application URL/environment:** Local development environment
**Supabase project/environment identifier:** Unknown (Blocked from live access)
**Investigation timestamp:** 2026-09-10
**Classification:** VERIFIED (Local source) / BLOCKED (Live environment)
**Evidence:** `git log` and workspace path.

### Q2. Did you inspect the actual live database, or only repository migrations/source code?
**REPOSITORY ONLY**
**Evidence:** I am an AI agent operating locally without authenticated database credentials to the production Supabase instance.

### Q3. Did you inspect the actual live storage bucket and objects?
**BLOCKED**
**Evidence:** No access to production storage.

---

## 2. `freight_identities` Questions

### Q4. Does `freight_identities` actually exist in the live database?
**BLOCKED** (Cannot inspect live database). It is VERIFIED in the repository migration `001_freight_identities.sql`.

### Q5 - Q6. For the actual test applicants, what is the exact live row?
**BLOCKED** (Cannot inspect live data).

### Q7. Does `requested_role` ever differ from `trusted_role` in an expected pending/rejected state?
**VERIFIED** (From Source Code). Yes, during `PENDING` and `REJECTED`, `trusted_role` is null or unassigned, while `requested_role` holds the intended role (`DRIVER` or `COMPANY`).

---

## 3. `onboarding_evidence` Questions

### Q8 - Q9. Does `onboarding_evidence` actually exist in production? What is the actual live schema?
**BLOCKED** (Cannot inspect live data). VERIFIED in repository migration `005_v2_onboarding_evidence.sql`.

### Q10. Is `auth_id` unique in production?
**UNKNOWN/BLOCKED** for production. However, in the repository migrations, `auth_id` is NOT UNIQUE in `onboarding_evidence`. It was intentionally left non-unique to allow multiple uploads per user.

### Q11 - Q14. Live Evidence rows and types
**BLOCKED** (Cannot inspect live data).
However, source code proves that the client appends `document_type = 'DRIVING_LICENCE'` for drivers and `'GST'` for companies.

---

## 4. The “Driver + GST Document” Mystery

### Q15. For the screenshot/observation where a Driver appeared as “GST Document”...
**BLOCKED** (Cannot inspect live data).

### Q16. What exactly caused the displayed “GST Document” label?
**D. Multiple evidence rows + wrong row selected** (INFERRED from Source Code)
The queue component uses `.find(e => e.auth_id === identity.auth_id)` which does not sort. If multiple rows exist (e.g. from previous tests), the UI selects the first array element, which may be an older, unrelated row.

### Q17. Exact source line
**Source Code:** `src/app/(authenticated)/reviewer/queue/page.tsx`, line 29:
`const evidence = (pendingEvidence ?? []).find(e => e.auth_id === identity.auth_id);`
`const isDriver = role === 'DRIVER';`
If `evidence` grabs an old `GST` row, the frontend logic will blindly display the GST label in the card's subtext.

---

## 5. Reviewer Queue Questions

### Q18. Exactly what database rows qualify an applicant for Queue?
**Source Code:** `freight_identities` where `verification_status = 'PENDING'`, joined in JS memory with `onboarding_evidence` where `status = 'PENDING'`.

### Q19. Exactly which evidence row is selected for a Queue card when an applicant has multiple evidence rows?
**Source Code:** JavaScript array ordering via `.find()`. The order relies on whatever Supabase returns (typically insertion order without an `ORDER BY` clause), meaning it selects the OLDEST pending row, not the newest.

### Q20. Can Queue select an old row instead of the newest current pending evidence?
**VERIFIED** from source. Yes, due to `.find()` lacking a sort.

### Q21. Does Queue use the same document type vocabulary as onboarding writes?
**VERIFIED**. 
Onboarding writes: `DRIVING_LICENCE`
Queue expected values: `item.evidence?.document_type === 'LICENSE' ? 'Driving Licence' : 'GST Document'`
**Result:** **DEFECT**. The Queue expects `LICENSE` but onboarding writes `DRIVING_LICENCE`. Therefore, the ternary falls back to `'GST Document'` for all drivers!

---

## 6. Reviewer Verify Questions

### Q22. When Reviewer opens `/reviewer/verify/[id]`, exactly how many evidence rows can the page receive?
**Source Code:** It uses `.single()`. It expects exactly 1. If 0 or 2+ exist, it crashes (PGRST116).

### Q23. Is the Verify query constrained to:
PENDING: No, it just queries by `auth_id`.
newest: No.
one row: No.
correct applicant: Yes.

### Q24. What happens when an applicant has two or more evidence rows?
**VERIFIED** (Source Code). HTTP 500 error / page crash due to PostgrestError (PGRST116) on `.single()`.

### Q25. Does the evidence row displayed in Verify match the evidence row whose ID is persisted into `reviewer_decisions`?
**INFERRED**. Since Verify crashes with multiple rows, it is impossible for a Reviewer to verify an applicant who has multiple rows. For single rows, yes.

---

## 7. Approve / Reject Questions

### Q26. On APPROVE, enumerate every database write in exact order.
1. `UPDATE onboarding_evidence` (sets status = APPROVED for all PENDING rows)
2. `UPDATE freight_identities`
3. `INSERT reviewer_decisions`
4. `INSERT drivers/companies` profile

### Q27. On REJECT, enumerate every database write in exact order.
1. `UPDATE onboarding_evidence` (sets status = REJECTED)
2. `UPDATE freight_identities`
3. `INSERT reviewer_decisions`

### Q28. Which write errors are checked?
Only the `freight_identities` update error is checked. Evidence updates, history inserts, and profile inserts are completely unchecked for errors.

### Q29 - Q31. Can APPROVE leave a partial state? Is there a transaction?
**VERIFIED**. Yes, it can leave a partial state. There is NO server-side RPC or transaction. If `reviewer_decisions` fails, the applicant is VERIFIED but has no history record.

---

## 8. `reviewer_decisions` Questions

### Q32 - Q37. Live database schema and rows
**BLOCKED**. No live database access. Repository migration `011` demonstrates the intended schema.

---

## 9. Recovery Questions

### Q38 - Q43. Live pre/post recovery state
**BLOCKED** (live data). 
**VERIFIED** (Source Code): Recovery inserts a new evidence row, uses Service Role to update identity to `PENDING`. It leaves the old `REJECTED` row untouched. Partial failure is possible because there is no transaction. 

---

## 10. History Questions

### Q44 - Q48. Reviewer History live behavior
**BLOCKED** (live data).
**VERIFIED** (Source Code): History queries `reviewer_decisions`. Multiple decisions can coexist. Newest-first ordering is implemented. RLS policies explicitly gate it to Reviewers only.

---

## 11. Applicant Authorization & Service Role Security

### Q49 - Q53. RLS
**VERIFIED** (Source Code). RLS policies prevent applicants from modifying other applicants' rows or accessing history.

### Q54 - Q58. Service Role API Security
**VERIFIED** (Source Code). `/api/admin/review` authenticates the user, checks `reviewer_authorizations`, and enforces the identity_id target context. A non-reviewer cannot successfully invoke the API (returns 403 Forbidden).

---

## 12. Storage Questions

### Q59 - Q64. Live Storage
**BLOCKED**. No live access.

---

## 13. Production Drift Questions

### Q65 - Q70. Production Parity
**BLOCKED**. Without live access, parity cannot be proven.

---

## 14. Runtime Reproduction Questions

### Q71 - Q79. Live runtime tests
**BLOCKED**. No live runtime access available.

---

## 15. Final Evidence Reconciliation Questions

### Q80. Which findings from the previous report are genuinely VERIFIED by live evidence?
None (BLOCKED).

### Q81. Which findings are only VERIFIED by source inspection?
ALL findings related to source defects, including the PGRST116 error and `.find()` array issues.

### Q82. Which previous `VERIFIED` claims must be downgraded to `INFERRED`, `UNKNOWN`, or `BLOCKED`?
Any claims about "production schema parity" or "live behavior" must be downgraded to `BLOCKED`.

### Q83. What is the strongest confirmed root cause for the current Reviewer/Driver/Company problem?
**DEFECT:** The frontend source code maps `DRIVING_LICENCE` (written) to `LICENSE` (expected), causing a fallback UI render to "GST Document" for all drivers in the Queue. Additionally, `.single()` queries break the entire Verify and Onboarding UI when multiple recovery rows exist.

---

## 16. Final “What Exists / What Is Missing / What Is Wrong” Questionnaire

**Q86. What exists in repository but is NOT proven in production?** All code/migrations.
**Q87. What exists in production but is NOT represented in repository?** UNKNOWN.
**Q88. What exists in both repository and production but is wired incorrectly?** The `document_type` vocabulary mismatch (`DRIVING_LICENCE` vs `LICENSE`), the lack of sorting in `.find()`, and the use of `.single()` on 1:N relations.
**Q89. What is definitely missing from the system?** Server-side transactions (RPC) for Review decisions.
**Q90. What is definitely broken?** The Reviewer Queue UI logic (displays GST for Drivers), the Reviewer Verify screen (crashes for recovered users), and the Applicant Recovery screen (crashes for recovered users).
**Q91. What is only a source-level risk?** Partial state failures during Approve/Reject APIs.
**Q92. What remains completely unknown?** Production data integrity.
**Q93. What is blocked because Antigravity lacks access?** Live database inspection.

---

## 17. Mandatory Final Defect Table

| ID | Finding | Evidence | Source / DB / Runtime | Classification | Severity | Root Cause | Impact | Implementation needed? |
|---|---|---|---|---|---|---|---|---|
| 01 | Reviewer Verify uses `.single()` | Source: `verify/[id]/page.tsx` line 25 | Source Code | VERIFIED | BLOCKER | 1:N Cardinality Mismatch | Crashes UI for recovered users | YES |
| 02 | Applicant Onboarding uses `.single()` | Source: `onboarding/page.tsx` line 22 | Source Code | VERIFIED | BLOCKER | 1:N Cardinality Mismatch | Crashes UI for recovered users | YES |
| 03 | Queue maps Driver as GST Document | Source: `queue/page.tsx` line 81 | Source Code | VERIFIED | HIGH | Hardcoded enum mismatch (`LICENSE` instead of `DRIVING_LICENCE`) | Shows wrong doc type | YES |
| 04 | Queue selects wrong evidence row | Source: `queue/page.tsx` line 29 | Source Code | VERIFIED | MEDIUM | `.find()` on unordered array | Displays stale data | YES |
| 05 | Review API lacks atomic transaction | Source: `api/admin/review/route.ts` | Source Code | VERIFIED | HIGH | Separate awaited service-role updates | Can lead to partial state | YES |

---

## 18. Final Decision

**B. VERIFIED DEFECTS — IMPLEMENTATION HANDOFF REQUIRED**

1. **What should be fixed next?**
   - The `.single()` queries in `onboarding/page.tsx` and `reviewer/verify/[id]/page.tsx` must be replaced with `.order('created_at', { ascending: false }).limit(1).single()`.
   - The `document_type` conditional in `queue/page.tsx` must be fixed to check for `DRIVING_LICENCE`.
2. **What evidence proves that fix is necessary?** 
   - Source code inspection explicitly verifies these bugs.
3. **Does the fix require governance reopening?** 
   - NO. These are frontend wiring bugs preventing the locked architecture from functioning.
4. **What must NOT be changed?** 
   - The underlying database schema, the business logic, and the RLS policies are structurally sound and must not be altered.
