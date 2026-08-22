# Freight / Delivery-Track — Authentication Redesign Investigation

## Purpose

Investigate the current authentication implementation before any authentication redesign is implemented.

This is an **INVESTIGATION-ONLY** instruction.

## Strict rules

- Do NOT modify source code.
- Do NOT modify database schema.
- Do NOT modify RLS policies.
- Do NOT modify Supabase Auth settings.
- Do NOT modify Vercel settings.
- Do NOT modify environment variables.
- Do NOT deploy.
- Do NOT run migrations.
- Do NOT fix anything.
- Do NOT create implementation code.
- Do NOT change the currently working Arrival → Check-in → Departure → Timeline → AI Evidence Summary flow.

Follow the project investigation discipline:

**OBSERVATION → INVESTIGATION → EVIDENCE → ROOT CAUSE / GAP → DECISION INPUT**

Tag findings as **VERIFIED / INFERRED / UNKNOWN**.

## 1. Current project/auth source investigation

Inspect the actual source repository and identify:

- Next.js / React versions.
- Supabase packages and versions.
- Auth client/server setup.
- Login/signup/logout files.
- Middleware/proxy/session handling.
- Protected routes.
- Server actions/route handlers involved in auth.
- How the current authenticated driver is resolved.
- How `drivers.auth_id` is currently used.
- Relevant environment variable names only; never expose secret values.

Trace the real current flow:

**UI → auth call → Supabase Auth → session → driver resolution → Dashboard**

## 2. Current database/auth relationship

Inspect the actual Supabase records/schema available to the project and report the current structure of the `drivers` table, especially:

- `id`
- `driver_code`
- `auth_id`
- `email`
- constraints
- unique indexes
- foreign keys
- nullable/non-nullable fields
- relevant relationships to `trips` and event/evidence data

Determine the actual relationship between:

`drivers.auth_id` ↔ `auth.users.id`

Also determine whether `driver_code` is currently generated, stored, unique, hardcoded, or used by the application.

Do not expose passwords, tokens, service-role keys, or other secrets.

## 3. Current Supabase Auth investigation

Verify the current behavior/configuration relevant to:

- email/password signup
- email/password login
- email confirmation
- logout
- session persistence
- redirect URLs if accessible
- current Site URL / allowed redirect configuration if accessible

Determine what is already working and what is only assumed.

## 4. Current RLS/security investigation

Inspect RLS policies relevant to:

- `drivers`
- `trips`
- `events`
- evidence/photo-related data

For each relevant policy report:

- table
- policy name
- operation
- role
- USING condition
- WITH CHECK condition
- what the policy actually permits

Pay particular attention to whether a logged-in driver can access only their own driver/trip/event data.

Do not change policies.

## 5. Locked target authentication design to investigate

The intended driver experience is:

### First-time registration

1. Driver chooses **Create Account**.
2. Driver enters:
   - Email ID
   - Password
3. Account is created through Supabase Auth.
4. The system has/creates a unique Driver ID such as:
   - `DRV001`
   - `DRV002`
5. The Driver ID is associated with exactly that driver's account/email.
6. The Driver receives the Driver ID through the registered email.
7. Driver enters the received Driver ID to continue to the Dashboard.
8. The system verifies that the entered Driver ID belongs to the authenticated/verified account.
9. Only then is Dashboard access granted.

### Returning driver login

After logout, the driver should enter:

**Driver ID + Password**

The system must verify that:

**entered Driver ID ↔ associated driver account/email ↔ password**

all correspond to the same driver account before allowing Dashboard access.

A wrong Driver ID or wrong password must result in no Dashboard access.

## 6. Architecture question to investigate

Investigate whether the following architecture is correct and safe with the current implementation:

```text
Freight / Delivery-Track UI
        │
        │ Driver ID + Password
        ▼
Freight Auth Layer (Next.js Server)
        │
        │ driver_code
        ▼
      drivers
   ┌────┼───────────┐
   │    │           │
   │ driver_code    │ auth_id
   │    │           │
   │    │           ▼
   │    │       auth.users
   │    │           │
   │    └── email ─┘
   │
   ▼
Supabase Auth
        │
     Session
        │
        ▼
   Dashboard
```

Determine whether `driver_code` should be treated as application-level identity while Supabase Auth continues to use email/password internally, or whether the current project requires a different architecture.

Do not implement the answer yet.

## 7. Security questions

Investigate specifically:

- Can a Driver ID safely be used as a login identifier through a server-side lookup/bridge?
- Where should Driver ID → email/auth identity resolution occur?
- Could the current/client-side approach expose another driver's email?
- Could Driver IDs be enumerated?
- Could one driver enter another driver's Driver ID and gain access?
- Does the current session guarantee that the Driver ID belongs to the authenticated user?
- Are `auth.uid()` checks correctly used where needed?
- Can a driver access another driver's trips/events by manipulating IDs or URLs?
- Is any service-role credential exposed to the browser?
- What protections are required around signup, Driver ID assignment, and login?

## 8. Email / Driver ID delivery investigation

Investigate the current project and Supabase email setup for the requirement that a newly created driver's unique Driver ID be delivered to that driver's registered email.

Determine:

- What email mechanism is currently available.
- Whether Supabase's existing email flow can support the required Driver ID delivery.
- Whether a custom email provider/domain is eventually required.
- What configuration changes would eventually be needed.

Do not configure or send anything during this investigation.

## 9. Existing working-flow protection

Verify how authentication currently connects to the already-working MVP:

**Arrival → Check-in → Departure → Timeline → AI Evidence Summary**

Identify exactly what authentication changes could affect:

- driver resolution
- active trip resolution
- event insertion
- event ownership
- timeline reads
- AI evidence reads

The investigation must explicitly identify what can remain unchanged.

## 10. Required report

Create the investigation report in:

`03_IMPLEMENTATION/implementation_reports/Chat7_Node3_Report_AuthenticationRedesignInvestigation.md`

The report must contain:

1. Executive Summary
2. Current Authentication Architecture
3. Current `drivers` / `auth.users` Relationship
4. Current Supabase Auth Configuration
5. Current RLS / Ownership Model
6. Current Login/Signup/Logout Flow
7. Current Driver ID Handling
8. Current Email Handling
9. Security Findings
10. Target Design vs Current Design
11. Gaps / Risks
12. What Can Be Reused
13. What Must Change Later
14. Recommended Architecture (investigation conclusion only)
15. Recommended Implementation Sequence
16. Files Likely Requiring Changes
17. UNKNOWN / Evidence Still Needed

For every important claim, include the evidence source (file path, relevant code location, SQL/schema evidence, configuration evidence, or command output).

## 11. Completion boundary

When the investigation is complete:

- Save the report in the exact path above.
- Do not implement anything.
- Do not push source-code changes.
- Report the investigation status and the report path.
- Wait for the reasoning brain / Ayush to review the report before any implementation instruction is created.

**This task ends at investigation.**
