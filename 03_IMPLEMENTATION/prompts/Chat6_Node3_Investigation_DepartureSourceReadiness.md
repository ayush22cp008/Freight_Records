# Freight — Chat6 Node3 Investigation: Departure Source Readiness

## Purpose

Perform a **read-only source investigation** of the current Freight application to determine implementation readiness of **Event 3 — Departure**.

This investigation is the next step after the personally verified workflow:

**Arrival → Check-in → Departure**

Arrival and Check-in are already implemented and verified. Do not modify them.

## Current State

- Arrival: implemented and personally verified by Ayush.
- Check-in: implemented and personally verified by Ayush.
- Dashboard now shows **Check-in Complete → Start Departure**.
- Departure is the next and final event in the core trip workflow.
- Existing Arrival and Check-in infrastructure should be reused wherever appropriate.

## Locked Departure Requirements

Departure must follow the established event pattern and must:

1. Capture foreground GPS.
2. Obtain an authoritative server timestamp.
3. Require a photo as evidence.
4. Insert an immutable `departure` event.
5. Set `event_type = 'departure'` server-side.
6. Prevent duplicate Departure for the same trip using the existing event constraint/pattern.
7. Complete the trip workflow after successful Departure.
8. Preserve Arrival and Check-in behavior.

## Investigation Tasks

Inspect the actual current source and determine:

### 1. Existing event infrastructure

Identify exact reusable paths for:
- GPS capture.
- Server timestamp.
- Photo capture/upload.
- Event insertion.
- Authenticated driver resolution.
- Existing duplicate handling.

Determine whether the Check-in implementation already provides a suitable direct pattern for Departure.

### 2. Dashboard / Trip Hub

Inspect the current Dashboard and verify:
- `hasArrival`, `hasCheckin`, and `hasDeparture` state logic.
- The `Start Departure` CTA.
- Destination route expected by Dashboard.
- What state is shown after `hasDeparture` becomes true.

Do not modify Dashboard during this investigation.

### 3. Existing Departure source

Search for:
- `/events/departure`
- `event_type = 'departure'`
- departure API routes
- departure components
- departure database logic

If none exist, state that clearly.

### 4. Database/schema

Verify:
- `departure` is already an allowed event type.
- Existing uniqueness constraint prevents duplicate departure.
- Photo field supports the required departure photo.
- No schema change is required.

Do not modify schema.

### 5. Workflow/state completion

Determine what the Dashboard does after Departure succeeds and whether the current state machinery already supports final-trip completion.

## Evidence Rules

Classify every finding as:

- **VERIFIED** — directly confirmed from current source.
- **INFERRED** — reasonable conclusion from inspected source.
- **UNKNOWN** — cannot be established.

Do not guess.

## Scope Restrictions

Do NOT:

- Implement Departure.
- Modify source code.
- Modify Arrival.
- Modify Check-in.
- Modify Dashboard.
- Modify database/schema/RLS.
- Add unrelated features.
- Fix unrelated security/architecture issues.

## Required Report

Create the report in:

`03_IMPLEMENTATION/implementation_reports/`

Use:

`Chat6_Node3_Report_DepartureSourceReadiness.md`

Include:

1. Investigation Objective
2. Source Snapshot / Commit or Working-State Identifier
3. Files Inspected
4. Reusable Infrastructure Findings
5. Dashboard / Workflow Findings
6. Existing Departure Findings
7. Database / Schema Findings
8. VERIFIED / INFERRED / UNKNOWN table
9. Blockers / Risks
10. Departure Implementation Readiness Decision
11. Smallest Safe Implementation Surface — exact files likely to add/change, without implementing
12. Recommended Next Step for ChatGPT/Ayush

**Investigation only. No source-code changes.**
