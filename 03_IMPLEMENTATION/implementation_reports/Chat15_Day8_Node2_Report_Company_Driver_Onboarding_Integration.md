# Chat 15 (Day 8): Node 2 Company/Driver Onboarding Integration Investigation Report

## 1. Executive Summary
This investigation reviewed the integration between the recently implemented Node 2 authentication boundary (`freight_identities`) and the existing application logic (`drivers` table and the authenticated dashboard). The core finding is that while the new Identity mechanism and Active Gate correctly protect the application (blocking PENDING users), the post-gate application retains legacy assumptions about identity. Specifically, the dashboard still directly queries the `drivers` table using `auth_id`, leading to a "No Driver Profile" error for properly verified Node 2 users. Furthermore, the role selection and evidence submission workflows described in the Node 2 contract have not yet been implemented.

## 2. Root-Cause Validation
- **Finding A (Verified identity reaches legacy Driver page):** VERIFIED. The `src/app/(authenticated)/page.tsx` directly queries the `drivers` table via `auth_id`. Since the new signup flow intentionally stops inserting into `drivers`, this query fails and triggers the "No Driver Profile" empty state.
- **Finding B (Signup currently produces a Driver requested role):** VERIFIED. The Postgres Auth trigger `on_auth_user_created` correctly extracts `requested_role` from `raw_user_meta_data`, but defaults to `'DRIVER'`. Because `src/app/signup/page.tsx` does not yet present a role selector, this default is always applied.
- **Finding C (Document/evidence onboarding is not demonstrated):** VERIFIED. The UI currently has no step to collect evidence (Driving Licence for Drivers, GST for Companies) after signup.

## 3. Resolution of Architectural Questions

1. **What is the authoritative post-verification application identity boundary?** 
   `freight_identities` is the authoritative source for whether a user is active, verified, and what their canonical trusted role is.
2. **How should `freight_identities` map to role-specific records?** 
   A clear architectural decision is required. Options:
   - A: Update `drivers` and `companies` to reference `freight_identities(id)` instead of `auth.users(id)`.
   - B: Keep `auth_id` as the join key across all tables (since `freight_identities` has a 1:1 mapping with `auth.users`).
3. **Should the authenticated home/dashboard resolve application context from Freight Identity?** 
   Yes. The dashboard should first inspect the Freight Identity's `trusted_role` (e.g., Driver vs. Company) and route or render the appropriate role-specific view, rather than unconditionally querying the `drivers` table.
4. **Where does a user select `requested_role` during signup?** 
   The signup form (`src/app/signup/page.tsx`) must be updated to include a "Role Selection" (Company / Driver) which is passed in `supabase.auth.signUp()` as `data.requested_role`.
5. **What exact evidence is submitted and where is it stored?** 
   Requires Node 2 onboarding implementation. Needs an onboarding UI for PENDING users to upload files (to Supabase Storage) and a database tracking mechanism.
6. **What exact server-authorized reviewer workflow establishes `trusted_role`?** 
   Requires an admin UI or external API utilizing `service_role` to transition `verification_status` from `PENDING` to `VERIFIED`, and formally set `trusted_role`.
7. **What audit evidence is required for approval/rejection?** 
   Open question. Needs definition (e.g., timestamps, reviewer ID).
8. **How should a VERIFIED Company user enter the Company application area?** 
   Via role-based routing in the authenticated layout/dashboard (e.g., `if (identity.trusted_role === 'COMPANY') return <CompanyDashboard />`).
9. **How should a VERIFIED Driver user enter the Driver application area without Driver Code as auth credential?** 
   Via standard email/password. The `trusted_role` identifies them as a Driver, and the system queries their `drivers` application record using their `auth_id` or `identity_id`.
10. **Which parts of the current Driver-specific UI are legacy?** 
    The hardcoded assumption in `page.tsx` that *every* user is a Driver is legacy Node 1 behavior that must be replaced by a role-aware router.

## 4. Current State
- Auth signup = OBSERVED WORKING
- Freight Identity creation = OBSERVED WORKING
- Pending Verification gate = OBSERVED WORKING
- Post-gate Driver dependency = VERIFIED LEGACY INTEGRATION GAP
- Company/Driver signup selection = OPEN
- Driver/Company evidence submission = OPEN
- Reviewer workflow = OPEN
- Company/Driver application mapping = OPEN

## 5. Conclusion & Recommendation
No application code modifications have been made during this investigation. The codebase is safe and the Active Gate functions properly, but the user experience hits a dead-end at the dashboard. 

**Next Steps:** We require explicit design decisions from the Product/Architecture owner (Ayush) on the exact onboarding flow (evidence collection) and the specific database mapping (how `drivers` and `companies` link to the verified identity) before proceeding with the next implementation prompt.
