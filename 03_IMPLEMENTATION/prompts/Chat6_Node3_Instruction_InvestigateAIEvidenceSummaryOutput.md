# Freight — Chat6 Node3 Investigation: AI Evidence Summary Output Regression

## Task

Perform a **read-only investigation first**. Do NOT change application code.

The current AI Evidence Summary output has changed unexpectedly.

### Expected behavior

The AI summary should contain the factual information for all three deterministic events:

**Arrival → Check-in → Departure**

including the recorded timestamps and the other permitted evidence fields.

### Current behavior

The latest browser screenshot shows only a short fragment:

`On 2026-08-21T07:49:`

The previous browser output contained a much longer response covering Arrival, Check-in, and Departure, but it also exposed a `<think>...</think>` reasoning trace. The reasoning-trace repair was then implemented.

The latest result therefore suggests that the `<think>` removal itself is not enough to explain the regression. We need to determine why the final user-facing response is now incomplete.

## Evidence Already Established

The existing implementation report states:

- deterministic evidence is fetched server-side from the authenticated user's trip;
- the server validates Arrival + Check-in + Departure before calling Groq;
- the AI payload contains `type`, `server_timestamp`, `latitude`, `longitude`, `gps_accuracy`, and `photo_provided`;
- Groq model discovery is dynamic rather than a permanently hardcoded model;
- the server strips `<think>...</think>` before returning the response;
- build passes. 

Read these before investigating:

- `03_IMPLEMENTATION/implementation_reports/Chat6_Node3_Report_AIEvidenceSummaryImplementation.md`
- `03_IMPLEMENTATION/prompts/Chat6_Node3_Instruction_ImplementAIEvidenceSummary.md`
- `03_IMPLEMENTATION/prompts/Chat6_Node3_Instruction_FixGroqModelDiscovery.md`
- `03_IMPLEMENTATION/prompts/Chat6_Node3_Instruction_FixAIEvidenceSummaryOutput.md`

## Investigation Questions

Inspect the **actual current source code**, not only the reports.

Determine precisely:

1. What exact model is being selected at runtime right now?
2. What exact `max_tokens` / `max_completion_tokens` / output-token limit is configured, if any?
3. What exact system prompt is currently sent to Groq?
4. What exact user payload is currently sent to Groq?
5. Does the application request one generation response or perform any additional processing?
6. What exact raw text does the Groq response contain before sanitization?
7. What exact text remains after `<think>` sanitization?
8. Is the current short output caused by:
   - token/output limit;
   - model behavior;
   - prompt regression;
   - response parsing;
   - `<think>` sanitization;
   - UI rendering/truncation;
   - stale browser/dev-server state;
   - another implementation issue?
9. Does the server actually receive all three events immediately before the Groq call?
10. Does the AI payload contain all three events immediately before the Groq call?
11. Is the current model a reasoning model whose final answer is being truncated or otherwise affected by the configured generation parameters?
12. Is there any code that takes only the first line/sentence/token/substring of the response?

## Critical Comparison

Compare the current implementation against the behavior shown in the two supplied browser states:

### Previous output

The previous output visibly contained a complete three-event summary:

- Arrival
- Check-in
- Departure

It also visibly contained a `<think>` reasoning block.

### Current output

The current output visibly contains only:

`On 2026-08-21T07:49:`

We need to identify exactly what changed between these states and whether the change was intentional.

## Important Constraint

Do NOT assume the problem is the sanitizer.

The sanitizer is specifically intended to remove reasoning text and preserve the final answer. Verify its actual input and output before drawing conclusions.

Do NOT assume the database is missing events. Verify the server-side evidence payload.

Do NOT assume the model is broken. Verify the actual selected model and request parameters.

Do NOT modify:

- Arrival
- Check-in
- Departure
- Timeline event rendering
- database schema
- RLS
- authentication architecture
- Groq provider choice
- dynamic model discovery architecture

## Required Investigation Method

Use source inspection and, if safe, local runtime logging/debugging that does NOT expose the API key or sensitive credentials.

It is acceptable to temporarily inspect/log sanitized metadata such as:

- selected model ID
- event count
- event types
- prompt length
- payload event count
- raw response length
- sanitized response length

Do NOT log:

- `GROQ_API_KEY`
- full secrets
- authentication tokens
- unnecessary personal data

If raw model output must be inspected, sanitize/redact any sensitive data first.

## Required Report

Update/create:

`03_IMPLEMENTATION/implementation_reports/Chat6_Node3_Report_AIEvidenceSummaryOutputInvestigation.md`

The report must contain:

1. Investigation objective.
2. Current source files inspected.
3. Current runtime model selected.
4. Current generation/output-token configuration.
5. Current system prompt behavior.
6. Current deterministic evidence payload shape and event count.
7. Raw response behavior before sanitization.
8. Sanitized response behavior.
9. Root cause of the incomplete summary.
10. Evidence proving the root cause.
11. Whether the `<think>` sanitizer is functioning correctly.
12. Whether the three-event evidence gate is functioning correctly.
13. Exact minimal fix recommendation.
14. Files that would need to change for the fix.
15. Explicit statement that **no code was changed during this investigation**.

## Completion Boundary

This task is complete when we have a technically supported root cause and a minimal, precise repair recommendation.

**Do not implement the fix in this task. Investigation first.**
