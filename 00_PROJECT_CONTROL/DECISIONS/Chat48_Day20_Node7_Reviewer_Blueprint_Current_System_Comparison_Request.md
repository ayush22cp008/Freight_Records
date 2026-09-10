# Chat48 / Day20 / Node7
# Reviewer Blueprint vs Current System — Comparison Request

**Status:** APPROVED FOR COMPARISON  
**Purpose:** Establish whether the current Reviewer portal is aligned with, ahead of, behind, or partially different from the locked Reviewer Blueprint before formal portal lock.

## Scope

Compare the locked Reviewer Blueprint against the current verified Reviewer implementation and manual evidence.

Required classification for each relevant Blueprint requirement:

```text
VERIFIED   = current system demonstrably satisfies the Blueprint requirement
AHEAD      = current system provides additional behavior without violating the Blueprint
PARTIAL    = current system broadly satisfies the intent but differs from the Blueprint detail
GAP        = current system does not satisfy the Blueprint requirement
UNKNOWN    = evidence is insufficient to classify safely
```

The comparison must explicitly cover:

- Reviewer responsibility boundary
- Reviewer mental model
- Verification Queue
- Applicant Verification
- Evidence Examination
- Identity / Role Verified action
- Approve / Reject flow
- Decision failure handling
- Decision Result
- Verification History
- Read-only completed record
- Evidence type semantics
- Current evidence selection/determinism
- Reviewer navigation
- Driver and Company compatibility
- Rejection/re-upload recovery
- Evidence viewing and technical failure handling
- Blueprint-specific interaction details

Known current-system findings that must be included:

1. Driver evidence label mismatch was confirmed and has a separate approved fix prompt: `DRIVING_LICENCE` must display as `Driving Licence`.
2. Reviewer Queue currently uses a weaker evidence-row selection approach than the deterministic newest-PENDING selection used in onboarding/verification.
3. Reviewer decision API performs multiple sequential database operations; atomic transaction safety has not been established.
4. Queue interaction differs from the Blueprint detail about direct evidence access, while still routing to Applicant Verification for actual examination.
5. Recent Reviewer navbar and global text-visibility fixes are already manually observed working.

Do not silently change the Blueprint or current system while performing this comparison. Any conflict must be surfaced explicitly and left for a separate governance decision.

## Lock Decision Gate

Formal Reviewer portal lock should occur only after:

```text
Blueprint comparison complete
        ↓
All GAP items resolved or explicitly accepted
        ↓
PARTIAL items explicitly governed
        ↓
UNKNOWN items resolved where material
        ↓
Manual verification complete
        ↓
Reviewer portal lock decision
```
