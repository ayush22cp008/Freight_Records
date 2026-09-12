# Chat50 / Node 7 — DeliveryProof Branding Update Implementation Instruction

## Execution Status
APPROVED FOR IMPLEMENTATION

## Source of Truth
- Project: Freight — AI Builders Hackathon
- Node: Node 7 — AI + Final Integration + Demo
- Source repository: `ayush22cp008/freight_hackathon`
- Records repository: `ayush22cp008/Freight_Records`
- Preceding investigation: `05_DEBUGGING/investigations/Chat50_Node7_Investigation_DeliveryProof_Branding_Change_Surface_Report.md`

## Objective
Change the user-facing product brand from temporary `Freight` to the approved permanent name **DeliveryProof**.

This is a presentation/branding-only change. Do not alter product behavior, architecture, data, authentication, APIs, database objects, business rules, lifecycle semantics, evidence logic, or security controls.

## Required Preflight
Before editing, confirm:
- project root and current directory
- source repository and branch
- current git status
- current source commit

Stop and report if the working tree contains unexpected pre-existing changes or if the repository/path does not match the expected source project.

## Required Search
Perform a comprehensive repository search for `Freight` / `freight` occurrences and classify each occurrence as either:

1. USER-FACING BRANDING — eligible for replacement with `DeliveryProof`.
2. TECHNICAL/INTERNAL IDENTIFIER — preserve unchanged.

Do not make a bulk case-insensitive replacement.

## Known User-Facing Branding Surfaces
The investigation already confirmed direct presentation-layer branding in:

- `src/app/(authenticated)/Navbar.tsx`
- `src/app/login/page.tsx`
- `src/app/(authenticated)/company/profile/page.tsx`
- `src/app/(authenticated)/company/history/page.tsx`
- `src/app/(authenticated)/company/created/page.tsx`
- `src/app/(authenticated)/company/incoming/page.tsx`

Also inspect all other portal and public/shareable surfaces for equivalent user-visible branding, including Driver, Company, Reviewer, authentication, onboarding, completion/history, and public evidence/share pages.

## Protected Technical Examples
Do NOT rename solely for branding:
- repository names (`freight_hackathon`, `Freight_Records`)
- package/project technical identifiers such as the package name `freight`
- `FreightIdentity`
- `freight_identities`
- API/backend/internal identifiers
- migration names and database/schema identifiers
- routes, environment/configuration identifiers, or storage keys unless proven to be user-facing branding
- historical Records references and historical filenames

## Implementation Rules
- Replace only confirmed user-facing branding.
- Preserve wording meaning when changing strings; make natural UI phrases such as `Freight Login` become `DeliveryProof Login`.
- Do not change functional labels where `freight` describes a domain concept rather than the brand unless the specific occurrence is clearly a brand label.
- Do not rename source/records repositories.
- Do not introduce unrelated UI/product changes.

## Verification
After the edits:
1. Run a targeted search to confirm intended user-facing `Freight` branding has been removed/replaced.
2. Confirm protected technical identifiers remain unchanged.
3. Run the project build.
4. Run the relevant existing automated tests/checks available in the source repository.
5. Report all files modified and any unexpected changes.

## Ayush Manual Verification Handoff
Do not declare the task fully verified solely from build/test results. Provide the changed surface list for Ayush to manually inspect the deployed application, including:
- login
- authenticated navbar/branding
- Driver portal
- Company portal
- Reviewer portal
- onboarding/profile/history/completion surfaces
- public/shareable evidence surface

## Scope Boundary
No GitHub push is authorized by this instruction. After successful build/test and Ayush manual verification, use the project's separate reusable commit/push instruction only with Ayush's explicit permission.
