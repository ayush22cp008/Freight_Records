# Chat50 Day21 Node7 — UI Header and Dashboard Title Consistency Implementation Prompt

## 1. Objective

Apply a narrowly scoped frontend UI consistency update across the already-implemented Driver, Company, and Reviewer dashboards.

This work is presentation-only and must not alter backend behavior, authentication/authorization rules, APIs, database/schema, RLS, trip lifecycle semantics, claiming, evidence, AI behavior, or Reviewer authority.

## 2. Requested UI Changes

### Driver Portal

- Keep the existing Driver dashboard functionality unchanged.
- Ensure the authenticated header visibly includes a `Sign out` control using the already-existing sign-out mechanism.
- Keep the dashboard page title explicitly as `Driver Dashboard`.

### Company Portal

- Keep the existing Company dashboard functionality unchanged.
- Ensure the authenticated header visibly includes a `Sign out` control using the already-existing sign-out mechanism.
- Change the main dashboard heading from generic `Dashboard` to explicit `Company Dashboard`.

### Reviewer Portal

- Keep the existing Reviewer `Sign out` control and behavior unchanged.
- Use an explicit `Reviewer Dashboard` page title/heading for the Reviewer dashboard.
- Preserve the existing Reviewer Queue and History navigation and all existing verification behavior.

## 3. Implementation Boundary

Allowed:

- Frontend page heading/title changes.
- Existing authenticated header presentation.
- Existing sign-out control presentation where the current sign-out mechanism already exists.
- Spacing/alignment/responsive presentation required to accommodate these UI elements cleanly.

Protected — do not change:

- Authentication implementation or session handling.
- Authorization or role resolution.
- RLS/security architecture.
- APIs/API contracts.
- Database/schema/data model.
- Trip lifecycle/state semantics.
- Claiming/marketplace behavior.
- Evidence requirements/integrity.
- AI behavior.
- Persistent Reviewer state.
- Reviewer authority/responsibility.

Do not invent a new sign-out mechanism. Reuse the existing authenticated sign-out behavior.

## 4. Locked-Portal Constraint

Driver, Company, and Reviewer portals are already accepted/locked. This is a narrowly scoped presentation correction for cross-portal consistency.

Do not make unrelated design, navigation, feature, or backend changes.

Do not modify any locked workflow beyond the explicitly listed UI presentation changes.

## 5. Acceptance Criteria

The implementation passes only if all of the following are true:

1. Driver dashboard visibly shows `Driver Dashboard` and a usable `Sign out` control.
2. Company dashboard visibly shows `Company Dashboard` and a usable `Sign out` control.
3. Reviewer dashboard visibly shows `Reviewer Dashboard` and retains the existing `Sign out` control.
4. Sign out still uses the existing authentication/sign-out behavior.
5. Existing Driver, Company, and Reviewer functionality remains unchanged.
6. No backend/API/database/RLS/auth/business-rule changes are introduced.
7. Desktop and mobile layouts remain visually usable with no obvious overflow or clipping introduced by the header/title changes.
8. No unrelated files or behavior are modified.

## 6. Execution Requirements for Antigravity

Before editing:

- Confirm the correct project root.
- Confirm current branch and repository.
- Confirm the target source files for Driver, Company, and Reviewer dashboards.
- Inspect existing sign-out implementation before changing UI.
- Stop and report if the requested UI change would require changing protected authentication/backend behavior.

During editing:

- Make the smallest possible frontend-only changes.
- Do not refactor unrelated code.
- Do not alter API calls or state semantics.

After editing:

- Run the relevant build/checks.
- Inspect desktop and mobile rendering.
- Verify Driver, Company, and Reviewer sign-out presentation.
- Verify dashboard titles.
- Confirm there are no console/runtime errors introduced by the change.
- Report exact files changed and any unexpected findings.

## 7. Verification Gate

Antigravity must return implementation evidence through the Records workflow.

Ayush remains the final manual tester and acceptance authority.

Do not declare this UI update accepted until Ayush manually verifies the three dashboards on the deployed application.

## 8. Scope Conclusion

This is a UI-only consistency update. No functional or backend change is authorized under this prompt.
