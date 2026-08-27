# Chat14 / Day 7 — Node 2 Authentication + Identity Implementation

## Execution boundary

Implement **Node 2 — Authentication + Identity** against the current `freight_hackathon` `main` baseline.

This instruction is the implementation bridge for Antigravity. Do not redesign the product, reopen locked Node 2 decisions, or implement later Nodes.

## Current verified baseline

The local source tree was reconciled on Day 7 and verified equal to `origin/main`. The current source baseline contains:

- `drivers` as the current application table with `driver_code`, `name`, and `auth_id`.
- `trips.driver_id -> drivers.id`.
- RLS enabled on `drivers` and `trips`.
- Migration `003_add_auth_id_to_drivers.sql` linking `drivers.auth_id` to `auth.users(id)`.
- Supabase SSR server/client helpers.
- A service-role server client in `src/lib/supabase-server.ts`.
- Email/password login.
- Driver-Code-based signup that creates an Auth user and then links `drivers.auth_id`.
- Authenticated layout that currently checks only `supabase.auth.getUser()`.
- No `freight_identities` table yet.

Do not reintroduce the discarded Driver-ID / auto-generated Driver-Code login experiment.

## Mandatory repository instruction

Before modifying code, read the repository's `AGENTS.md` and follow its Next.js instruction. It explicitly requires reading the relevant installed Next.js guide under `node_modules/next/dist/docs/` before writing code because this project uses a breaking Next.js version.

## Locked Node 2 contract

Implement the already-decided Node 2 Q1–Q7 contract. Do not reinterpret it.

### Identity model

Create the canonical `freight_identities` layer rather than treating `drivers` as the universal identity table.

The identity record must support the locked concepts:

- `auth_user_id` — Supabase Auth user UUID.
- `requested_role` — role requested during signup.
- `verification_status` — verification state.
- `trusted_role` — server-controlled trusted/approved role; never trust a client-provided role for authorization.
- Appropriate timestamps/identity metadata as required by the locked schema decision.

Use the locked database uniqueness invariant so one Auth user cannot acquire multiple Freight Identity rows.

### Auth-trigger invariant

Identity creation must be tied to Supabase Auth user creation using the locked Auth-trigger approach. Do not implement a fragile application-level sequence of “create Auth user, then independently insert identity” as the authoritative invariant.

The trigger must create the identity row from the Auth user creation event using safe server/database-controlled values and the approved default identity state.

### Signup

Adapt signup to the locked identity model while preserving the current email/password authentication model.

Signup must not allow the client to self-authorize a trusted role.

Do not use Driver ID as a login credential.

Do not use the discarded auto-generated Driver Code migration or login lookup flow.

Where the existing Driver Code is still required for the current Driver onboarding relationship, preserve that concept only where consistent with the locked Node 2 contract; do not make it the authentication identity.

### Login/session

Continue using Supabase Auth email/password authentication and the existing SSR session mechanism.

After authentication, resolve the canonical Freight Identity server-side from the authenticated Auth user ID.

Do not derive authorization from request-body role fields, UI state, local storage, or other client-controlled identity claims.

### Trusted role / verification gate

Implement the server-side identity/role gate defined by the locked contract:

- Authentication establishes the Auth user.
- Freight Identity establishes the application identity.
- Verification/trusted-role state controls whether the user is allowed into protected product functionality.
- A client must not be able to promote its own `trusted_role` or bypass verification.

Use the existing authenticated route boundary as appropriate, but do not assume that “has a Supabase session” means “authorized for all Freight functionality.”

### Server-side identity context

Create a small reusable server-side identity resolution mechanism appropriate to the current Next.js architecture. It should provide downstream protected routes with the authenticated Freight Identity rather than requiring every route to independently reconstruct identity semantics.

Keep this mechanism minimal and composable for Node 1 authorization work.

## Database safety requirements

- Add a new migration rather than rewriting historical migrations.
- Do not delete or mutate historical migration `001`, `002`, or `003` merely to make Node 2 fit.
- Apply constraints in the database for invariants that must not depend on application behavior.
- Do not add broad client-side RLS write policies that conflict with the locked RLS/service-role architecture.
- Do not expose the service-role key to browser/client code.
- Avoid SECURITY DEFINER behavior that is broader than necessary; scope privileges and search path safely according to the locked design and repository conventions.

## API/security requirements

- Validate request inputs server-side.
- Do not trust client-supplied `trusted_role`.
- Do not trust client-supplied identity IDs as proof of identity.
- Use the authenticated Supabase user/session as the root identity signal.
- Avoid identity-enumeration leaks in auth responses where the locked contract requires generic behavior.
- Preserve the existing password authentication mechanism unless a locked Node 2 requirement explicitly requires otherwise.

## Scope

This implementation covers Node 2 only:

1. Canonical Freight Identity persistence.
2. Auth-trigger identity creation.
3. Signup adaptation.
4. Login/session identity resolution.
5. Verification/trusted-role gate.
6. Reusable server-side identity context.
7. Required tests and verification evidence.

Do **not** implement:

- Company trip creation/publishing (Node 3).
- Driver marketplace/atomic claim (Node 4).
- Delivery tracking (Node 5).
- Security/evidence hardening beyond what is necessary to complete Node 2 (Node 6).
- AI/final integration (Node 7).

## Implementation sequence

Work in small, reviewable slices:

### Slice 1 — Database identity foundation

- Add the new migration for `freight_identities` and its required constraints/indexes.
- Add the Auth trigger/function according to the locked design.
- Ensure the migration is safe to apply and does not modify historical migrations.
- Validate the schema and trigger behavior with database-level tests or reproducible SQL verification.

### Slice 2 — Server identity resolution

- Add the reusable server-side identity resolver/context.
- Resolve by authenticated `auth_user_id`.
- Return a clear unauthenticated/no-identity/blocked state without exposing unnecessary internal details.

### Slice 3 — Signup integration

- Adapt signup to the new identity lifecycle.
- Preserve email/password Supabase Auth.
- Ensure the Auth trigger owns creation of the canonical identity row.
- Remove reliance on the old application-level identity-creation assumption.
- Preserve any required Driver relationship without making `drivers` the universal identity table.

### Slice 4 — Protected access gate

- Update the authenticated boundary and/or appropriate server route guard so protected Freight functionality requires the correct identity state from the locked contract.
- Do not overreach into Node 1's full authorization matrix; provide the identity/trust context Node 1 will consume.

### Slice 5 — Login/UI compatibility

- Keep email/password login.
- Remove no existing product capability unless required by the locked Node 2 contract.
- Do not reintroduce Driver-ID login.
- Update UI only as necessary to accurately represent the new identity/verification lifecycle.

## Required verification

Run appropriate repository checks, including at minimum the relevant lint/type/build/test commands available in the project.

Verify at the database/auth level that:

1. A newly created Auth user gets exactly one Freight Identity through the trigger.
2. The identity row references the correct Auth user.
3. Duplicate identity creation for the same Auth user is prevented by the database invariant.
4. Default requested/verification/trusted state matches the locked contract.
5. Client input cannot assign an arbitrary trusted role.
6. Login resolves the same canonical Freight Identity from the authenticated Auth user.
7. Protected routes reject unauthenticated users.
8. Protected routes do not treat an unverified/untrusted identity as fully authorized when the locked gate says otherwise.
9. Existing email/password authentication remains functional.
10. No service-role secret is exposed client-side.
11. Existing Node 1 RLS assumptions are not weakened.

## Evidence discipline

Do not report a requirement as VERIFIED without concrete evidence.

For each major acceptance item, report:

- command/query/test used,
- observed result,
- VERIFIED / INFERRED / UNKNOWN classification.

If a Supabase dashboard/manual configuration step is required and cannot be verified automatically, mark it UNKNOWN and state exactly what Ayush must manually verify.

## Stop conditions

Stop and report before making a broad redesign if:

- the locked Q1–Q7 contract cannot be implemented without changing an already-decided architecture;
- the current database state contradicts the migration assumptions;
- existing production/deployed data would require a migration strategy not covered by this instruction;
- the repository's actual Next.js version or installed APIs contradict the expected implementation path;
- unexpected local changes appear;
- a security-sensitive decision is required that is not covered by the locked contract.

Do not silently resolve any such conflict.

## Git workflow

Follow the project's GitHub/Antigravity workflow.

Do not assume implementation changes are pushed merely because local implementation succeeds.

The implementation report must identify:

- files changed/created,
- migration name,
- tests/checks run,
- relevant commit SHA if committed,
- whether changes were pushed,
- deployment status if applicable,
- manual verification still required.

## Completion boundary

Node 2 implementation is complete only after code implementation, automated checks, evidence, and the required manual verification checklist are documented.

Do not claim Node 2 COMPLETE merely because the code compiles.
