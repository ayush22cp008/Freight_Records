# Chat48 / Day 20 / Node 7 / Phase 1c
# Combined Onboarding Lifecycle Investigation Report

**Status:** INVESTIGATION COMPLETE  
**Project:** Freight — AI Builders Hackathon  
**Node:** Node 7 — AI + Final Integration + Demo  
**Phase:** Phase 1c  
**Chat:** Chat48  
**Day:** Day 20  
**Scope:** Fresh account signup → onboarding evidence submission → Pending Verification → Reviewer Queue → Reviewer Verify → rejection → rejected-applicant re-upload/recovery → Pending Verification  

---

## 1. Investigation Findings

The investigation evaluated the unexpected behavior where fresh signups (e.g., Driver or Company) immediately redirect to `/onboarding` and display the `Pending Verification` UI with `Evidence Type: Unknown`, skipping the evidence upload form entirely.

### 1.1 Fresh Account State

Analysis of `src/db/migrations/004_create_freight_identities.sql` confirms that a fresh account is created with:
```sql
verification_status text NOT NULL DEFAULT 'PENDING'
```
Consequently, a fresh signup is expected to create:
**A `PENDING` identity with no evidence yet.** 

### 1.2 Phase 1c Fix Impact

In the Phase 1c fix for rejected applicants, the gating logic on `src/app/(authenticated)/onboarding/page.tsx` was modified to:
```typescript
if (identity.verification_status === 'PENDING') {
```
The previous logic required the presence of evidence:
```typescript
if (evidence && identity.verification_status !== 'REJECTED') {
```

Because fresh accounts are created with `verification_status = 'PENDING'` by default, removing the requirement for `evidence` to be present causes the onboarding page to misinterpret a fresh account as an applicant who has already submitted evidence. Since there is no actual evidence row in `onboarding_evidence`, the UI renders with fallback values (e.g., `Evidence Type: Unknown`).

### 1.3 Reviewer Queue Absence

The Reviewer Queue correctly ignores these fresh accounts because it pairs `freight_identities` with `onboarding_evidence` rows. Since a fresh account has no corresponding pending evidence row, the `INNER JOIN` (or equivalent pair mapping) correctly filters them out, leaving the Reviewer Queue empty.

## 2. Root Cause Summary

The root cause is a cardinality and state assumption mismatch:
1. `verification_status = 'PENDING'` is the default state for **new, unsubmitted** applicants, not exclusively for **submitted, waiting for review** applicants.
2. The Phase 1c fix removed the `evidence` check on the onboarding page, assuming that any `PENDING` identity was waiting for review.

## 3. Recommended Fix

To resolve this issue while preserving the rejected applicant recovery flow, the `/onboarding` page must require BOTH a `PENDING` identity status AND the existence of a `PENDING` evidence row to display the "Pending Verification" UI.

**Recommended Code Change in `src/app/(authenticated)/onboarding/page.tsx`**:
```typescript
// Add the evidence check back to the gating logic
if (identity.verification_status === 'PENDING' && evidence) {
  return (
    // Pending Verification UI
  )
}
```

This change ensures:
- **Fresh accounts:** `verification_status = 'PENDING'`, `evidence = null` ➔ Displays "Complete Onboarding" form.
- **Submitted accounts:** `verification_status = 'PENDING'`, `evidence = {...}` ➔ Displays "Pending Verification".
- **Rejected accounts:** `verification_status = 'REJECTED'`, `evidence = null (or rejected row)` ➔ Displays "Re-upload Evidence" form.
- **Recovered accounts:** `verification_status = 'PENDING'`, `evidence = {...}` ➔ Displays "Pending Verification".
