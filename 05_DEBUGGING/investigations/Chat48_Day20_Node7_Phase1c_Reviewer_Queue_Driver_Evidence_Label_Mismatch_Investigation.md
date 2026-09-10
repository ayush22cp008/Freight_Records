# Chat48 / Day20 / Node7 / Phase1c
# Reviewer Queue — Driver Evidence Label Mismatch Investigation

**Status:** INVESTIGATION COMPLETE  
**Project:** Freight — AI Builders Hackathon  
**Node:** Node 7 — AI + Final Integration + Demo  
**Phase:** Phase 1c  
**Chat:** Chat48  
**Day:** Day20

## 1. Observation

During live Reviewer Queue testing, an applicant with:

```text
Claimed Role: DRIVER
```

was displayed as:

```text
GST Document
```

This contradicts the intended onboarding evidence semantics, where a Driver submits a Driving Licence.

## 2. Source Evidence

The current Reviewer Queue implementation queries pending onboarding evidence and formats the document label using this conditional:

```text
item.evidence?.document_type === 'LICENSE'
    ? 'Driving Licence'
    : 'GST Document'
```

Therefore any evidence type other than `LICENSE` is displayed as `GST Document`.

The established onboarding semantics use:

```text
DRIVING_LICENCE → Driver
GST             → Company
```

The current Queue mapping therefore does not recognize the actual Driver evidence type `DRIVING_LICENCE`.

## 3. Root Cause

**VERIFIED:** Reviewer Queue uses an incorrect evidence-type display mapping.

The Queue expects `LICENSE`, while the Driver onboarding flow uses `DRIVING_LICENCE`.

As a result:

```text
DRIVING_LICENCE
      ↓
not equal to LICENSE
      ↓
falls into fallback branch
      ↓
GST Document ❌
```

## 4. Expected Behavior

The Reviewer Queue must display:

```text
DRIVER + DRIVING_LICENCE → Driving Licence
COMPANY + GST            → GST Document
```

The stored evidence type itself must remain unchanged.

## 5. Scope Assessment

This is a narrow Reviewer presentation/mapping defect.

The appropriate fix is expected to be limited to the Reviewer Queue document-label mapping.

No database migration, evidence-schema change, onboarding API change, Reviewer decision logic change, or lifecycle-state change is justified by this evidence.

## 6. Preservation Requirements

Any subsequent fix must preserve:

- existing Driver onboarding evidence type `DRIVING_LICENCE`;
- existing Company evidence type `GST`;
- existing newest-PENDING evidence selection work;
- Reviewer Queue filtering behavior;
- Reviewer Verify behavior;
- Verification History;
- Driver and Company onboarding/recovery lifecycle;
- existing Reviewer navigation and styling.

## 7. Decision Boundary

No code change is authorized by this investigation alone.

A separate implementation instruction should authorize only the minimal mapping correction after explicit governance approval.

## 8. Conclusion

The observed Driver → GST Document display is a confirmed source-level bug. The root cause is the Reviewer Queue's incorrect `LICENSE` comparison rather than the actual stored Driver evidence type.
