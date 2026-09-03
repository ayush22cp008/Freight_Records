# Chat36 — Node 7 Phase 1a Company Public-Share UI Integration Instruction

## Objective
Inspect and, only if confirmed missing, minimally integrate the existing Phase 1a public-share management capability into the Company portal so a Company can create, copy, replace, and revoke a public evidence link for an eligible completed trip.

## Source of Truth
- `03_IMPLEMENTATION/plans/Chat36_Node7_Phase1a_Remediation_Verification_Updated_Plan.md`
- `03_IMPLEMENTATION/plans/Chat35_Node7_Phase1a_Public_Evidence_Sharing_Reconciled_Implementation_Plan.md`
- `02_ARCHITECTURE/Chat34_Node7_Phase1a_Public_Evidence_Sharing_Architecture_Finalization.md`
- Existing Phase 1a public-share implementation and APIs

## Inspect First
1. Inspect the current Company dashboard/trip-management UI and determine where Company users can access completed trips.
2. Inspect the existing public-share management API and its authorization/completion/evidence requirements.
3. Confirm whether the current UI already exposes this capability. Do not assume it is absent only from screenshots.

## If Missing
Add only the smallest Phase 1a-compliant Company UI integration using the existing public-share APIs.

Required Company capability for an eligible completed trip:
- Create public share.
- Display/copy the generated public URL.
- Replace an existing active share.
- Revoke an existing share.
- Show clear success/error states without exposing raw token values except the intended public URL.

## Security / Scope
- Company-only management; preserve server-side authorization.
- Do not expose this management capability to Drivers.
- Do not alter the existing Driver Trip/Timeline experience.
- Do not create a second public-share API or database model.
- Do not change token generation, hashing, privacy allowlist, or public verification behavior unless a verified defect requires it.
- Do not redesign the dashboard or begin Phase 1b.
- Do not add unrelated features or refactor unrelated code.

## Verification
- Test eligible completed-trip create/copy/revoke/replace flow.
- Test ineligible/non-completed trip behavior.
- Test Company authorization boundary.
- Confirm Driver cannot manage public shares.
- Confirm existing public `/share/[token]` behavior remains intact.
- Run relevant tests/typecheck.
- Update the existing `Chat35_Node7_Phase1a_Public_Evidence_Sharing_Implementation_Report.md` with actual results and any limitations.
- Commit and push changes.

## Stop Conditions
If the existing Company architecture does not provide a safe completed-trip entry point, or if implementation reality conflicts with the locked Phase 1a architecture, STOP and report the exact finding rather than inventing a new architecture.

After implementation and verification, STOP for Ayush manual verification/approval. Do not start Phase 1b and do not create Chat37.
