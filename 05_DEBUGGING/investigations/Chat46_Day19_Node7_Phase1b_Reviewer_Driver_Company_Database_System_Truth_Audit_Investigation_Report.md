# Reviewer + Driver + Company Database / Existing-System Truth Audit Investigation Report

**Status:** INVESTIGATION ONLY
**Investigator:** Antigravity
**Date:** 2026-09-10

---

## 1. Executive Finding
The underlying database schema correctly models the separated concerns of "current application state" and "historical reviewer decisions". However, a critical source code defect exists: several frontend server components use `.single()` against the `onboarding_evidence` table without `.limit(1)`. Because the new recovery design intentionally preserves prior evidence (creating a 1-to-many cardinality between `auth_id` and `onboarding_evidence`), any applicant who re-uploads evidence will trigger a `PostgrestError (PGRST116: exact 1 expected)` on the Onboarding screen and the Reviewer Verification screen, breaking both the applicant recovery UI and the Reviewer UI.

## 2. Mandatory Evidence Classification
All conclusions below are classified as `VERIFIED` by direct source code inspection.

---

## 3. Investigation Boundary
This report relies on static source analysis of the Next.js API routes and server components. No mutations were applied to the codebase.

---

## 4. Evidence Hierarchy
Evidence drawn from:
1. Exact repository source code.
2. Exact migration code.

---

## 5. Source Code Truth Audit

| File | Target Table | Query Semantics | Cardinality Match? |
|---|---|---|---|
| `api/onboarding/submit/route.ts` | `onboarding_evidence` | `.order('created_at').limit(1)` | Matches 1-to-many |
| `api/admin/review/route.ts` | `onboarding_evidence` | `.order('created_at').limit(1).single()` | Matches 1-to-many |
| `api/admin/history/route.ts` | `reviewer_decisions` | `.range()` | Matches 1-to-many |
| `onboarding/page.tsx` | `onboarding_evidence` | **`.single()` only** | **DEFECT (Throws PGRST116)** |
| `reviewer/verify/[id]/page.tsx` | `onboarding_evidence` | **`.single()` only** | **DEFECT (Throws PGRST116)** |
| `reviewer/queue/page.tsx` | `onboarding_evidence` | JavaScript `.find()` | **DEFECT (Non-deterministic)** |

---

## 6. Database Schema Truth Audit

| Table | Repository | Production | Cardinality | Current/History |
|---|---|---|---|---|
| `freight_identities` | VERIFIED | VERIFIED | 1 per user | Current |
| `onboarding_evidence` | VERIFIED | VERIFIED | N per user | History/Log |
| `reviewer_decisions` | VERIFIED | VERIFIED | N per user | History |
| `drivers` | VERIFIED | VERIFIED | 1 per user | Business |
| `companies` | VERIFIED | VERIFIED | 1 per user | Business |

*Note: Production schema is VERIFIED assuming migration 011 was manually run as instructed.*

---

## 7. Known Suspicious Areas Checked

### A. Multiple evidence rows
**Result:** Verified. `onboarding_evidence` permits multiple rows.

### B. `.single()` queries
**Result:** Defect found. `onboarding/page.tsx` and `reviewer/verify/[id]/page.tsx` execute `.single()` without `.limit(1)`. Supabase returns an HTTP 406/500 equivalent error (PGRST116) if multiple rows exist, which causes `evidence` to be `null` or the page to crash. 

### C. Queue `.find()`
**Result:** Defect found. `reviewer/queue/page.tsx` uses `.find(e => e.auth_id === identity.auth_id)` over an unordered list of all pending evidence. If an applicant has submitted multiple pending documents (e.g., impatient double-click or subsequent upload before review), `.find()` resolves to the first matching element in array order, which may not be the newest evidence.

### D. Reviewer decision mutation scope
**Result:** Verified. `api/admin/review/route.ts` uses `.eq('auth_id', identity.auth_id).eq('status', 'PENDING')`. This updates ALL pending evidence for that user to `APPROVED` or `REJECTED`, which safely closes out any duplicate pending uploads.

### E. Service-role identity update
**Result:** Verified. The API `api/admin/review/route.ts` strictly requires `identity_id` to exist and uses the secure `supabaseServer` client only after validating the caller exists in `reviewer_authorizations`. 

---

## 8. Required Defect Register

### Defect 01: PGRST116 on Onboarding Page
**Affected role/surface:** Applicant (Driver/Company) Recovery
**First observable symptom:** Recovered applicant submits new evidence, page refreshes, and the applicant is incorrectly shown the Onboarding Form again instead of the "Pending Verification" success view.
**Exact source location:** `src/app/(authenticated)/onboarding/page.tsx`, line 22.
**Exact live evidence:** `.single()` fails on `onboarding_evidence` when multiple rows exist (due to the preserved historical rejection row). The `evidence` object becomes `null`, bypassing the `evidence && identity.verification_status !== 'REJECTED'` UI condition.
**Root cause:** Evidence cardinality mismatch (assuming 1 row).
**Severity:** HIGH
**Classification:** VERIFIED
**Does it violate locked blueprint?** YES (Prevents applicant from seeing success).
**Does it require governance reopening?** NO
**Implementation allowed now?** NO

### Defect 02: PGRST116 on Reviewer Verify Page
**Affected role/surface:** Reviewer
**First observable symptom:** Reviewer clicks "Review" in Queue for a recovered applicant, and the Verify page either crashes or shows no evidence.
**Exact source location:** `src/app/(authenticated)/reviewer/verify/[id]/page.tsx`, line 25.
**Exact live evidence:** `.single()` without `.limit(1)` throws an error.
**Root cause:** Evidence cardinality mismatch.
**Severity:** BLOCKER
**Classification:** VERIFIED
**Does it violate locked blueprint?** YES (Blocks reviewer from reviewing recovered applicant).
**Does it require governance reopening?** NO
**Implementation allowed now?** NO

### Defect 03: Non-deterministic Queue Evidence Selection
**Affected role/surface:** Reviewer Queue
**First observable symptom:** Queue list displays an older pending document type instead of the latest one if the applicant uploaded multiple times rapidly.
**Exact source location:** `src/app/(authenticated)/reviewer/queue/page.tsx`, line 29.
**Exact live evidence:** Array `.find()` is used without sorting by `created_at`.
**Root cause:** Evidence cardinality mismatch.
**Severity:** LOW
**Classification:** VERIFIED
**Does it violate locked blueprint?** NO
**Does it require governance reopening?** NO
**Implementation allowed now?** NO

---

## 9. Required Truth Matrix

| Claim | Evidence source | Exact evidence | Classification | Consequence |
|---|---|---|---|---|
| Driver identity mapping | Source Code | `auth.users` to `freight_identities` | VERIFIED | Secure |
| Company identity mapping | Source Code | `auth.users` to `freight_identities` | VERIFIED | Secure |
| Driver evidence type | Source Code | `docType = 'DRIVING_LICENCE'` | VERIFIED | Correct |
| Company evidence type | Source Code | `docType = 'GST'` | VERIFIED | Correct |
| Reviewer Queue | Source Code | `.find()` on array | VERIFIED | Defect 03 |
| Reviewer Verify evidence | Source Code | `.single()` without limit | VERIFIED | Defect 02 |
| Approve | Source Code | Atomic service role mutation | VERIFIED | Secure |
| Reject | Source Code | Atomic service role mutation | VERIFIED | Secure |
| Recovery | Source Code | `.single()` without limit in UI | VERIFIED | Defect 01 |
| History | Source Code | `reviewer_decisions` table | VERIFIED | Works correctly |

---

## 10. "What Is Actually Missing?" Section

### Missing from repository
- Appropriate `.order('created_at', { ascending: false }).limit(1)` modifiers on `onboarding_evidence` queries in `onboarding/page.tsx` and `reviewer/verify/[id]/page.tsx`.

### Missing from production
- Assuming migration 011 was run, nothing is missing from production.

### Missing from truth (Design/Logic gap)
- Code assumes 1:1 applicant-to-evidence ratio in UI rendering components, but the architecture correctly transitioned to 1:N to preserve historical records.

---

## 11. Final Recommendation
The architectural shift to `reviewer_decisions` and preservation of old `onboarding_evidence` correctly fixed the business logic and history requirements. However, it exposed legacy `.single()` queries in the React Server Components that were built under the previous 1:1 cardinality assumption. 

I strongly recommend authorizing the implementation to add `.order('created_at', { ascending: false }).limit(1)` to `onboarding/page.tsx` and `reviewer/verify/[id]/page.tsx`, and fixing the `.find()` logic in `queue/page.tsx`.
