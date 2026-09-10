# Chat48 / Day 20 / Node 7 / Phase 1c
# Reviewer Queue — Driver Evidence Label Mismatch Investigation Report

**Status:** INVESTIGATION COMPLETE  
**Project:** Freight — AI Builders Hackathon  
**Node:** Node 7 — AI + Final Integration + Demo  
**Phase:** Phase 1c  
**Chat:** Chat48  
**Day:** Day 20  

---

## 1. Observation

During live Reviewer Queue testing, an applicant with a claimed role of `DRIVER` was displayed with the evidence label `GST Document`. This contradicts the intended onboarding evidence semantics, where a Driver submits a Driving Licence.

---

## 2. Source Evidence

Analysis of the Reviewer Queue source code (`src/app/(authenticated)/reviewer/queue/page.tsx` and related reviewer components) shows the following conditional formatting for the document label:

```typescript
item.evidence?.document_type === 'LICENSE'
    ? 'Driving Licence'
    : 'GST Document'
```

However, analysis of the applicant onboarding form (`src/app/(authenticated)/onboarding/OnboardingForm.tsx`) shows that the actual value submitted for a Driver's evidence is:

```typescript
const docType = isDriver ? 'DRIVING_LICENCE' : 'GST';
```

Therefore, the Queue maps any evidence type other than `LICENSE` to `GST Document`. Since the actual type is `DRIVING_LICENCE`, the mapping fails and falls back to the default `GST Document`.

---

## 3. Root Cause

**VERIFIED:** The Reviewer Queue uses an incorrect evidence-type display mapping.

The Queue components expect the string `'LICENSE'`, while the Driver onboarding flow correctly submits `'DRIVING_LICENCE'`. This literal string mismatch causes the conditional fallback to trigger, incorrectly labeling Driver evidence as `GST Document`.

---

## 4. Recommended Fix

Update the conditional mapping in all Reviewer-side components to check for `'DRIVING_LICENCE'` instead of `'LICENSE'`.

Files to update:
- `src/app/(authenticated)/reviewer/queue/page.tsx`
- `src/app/(authenticated)/reviewer/verify/[id]/ApplicantVerificationClient.tsx`
- `src/app/(authenticated)/reviewer/history/[id]/EvidenceViewerClient.tsx`

**Required change:**
```typescript
// Replace 'LICENSE' with 'DRIVING_LICENCE'
item.evidence?.document_type === 'DRIVING_LICENCE' ? 'Driving Licence' : 'GST Document'
```
