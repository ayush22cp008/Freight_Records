# Freight — Chat6 Node3 Repair Instruction: AI Evidence Summary Output Polish

## Task

Fix one specific remaining issue in the already-working **AI Evidence Summary** feature:

**The user-facing Timeline UI must display only the final factual summary, never the model's reasoning/thinking trace.**

The current browser verification shows the generated response contains a visible `<think>...</think>` block followed by the final summary. This is not acceptable user-facing output.

This is a **small output-polish/error-handling repair only**. Do not rebuild the AI Evidence Summary feature.

## Read First

Read:

- `03_IMPLEMENTATION/implementation_reports/Chat6_Node3_Report_AIEvidenceSummaryImplementation.md`
- `03_IMPLEMENTATION/prompts/Chat6_Node3_Instruction_ImplementAIEvidenceSummary.md`
- `03_IMPLEMENTATION/prompts/Chat6_Node3_Instruction_FixGroqModelDiscovery.md`

Preserve the existing Groq runtime model-discovery repair and deterministic evidence pipeline.

## Locked Architecture

Keep:

**Arrival → Check-in → Departure → Trip Complete → Timeline → AI Evidence Summary**

Keep:

- Groq API
- `GROQ_API_KEY`
- server-side API call
- runtime model discovery
- authenticated trip/evidence retrieval
- three-event gate
- one Groq generation call after model discovery
- existing Timeline UI structure

Do not modify Arrival, Check-in, Departure, Timeline event rendering, database schema, or RLS.

## Required Output Behavior

The final response returned to the browser must be **only the user-readable factual summary**.

The browser must never display:

- `<think>...</think>` blocks
- reasoning traces
- chain-of-thought
- model self-review/self-correction text
- internal planning text
- statements such as `Here's a thinking process:`
- internal verification/checklist text
- provider metadata

The user should see only the concise final factual summary.

## Server-Side Sanitization

Implement the smallest safe server-side output normalization before returning the summary to the client.

At minimum:

1. Read the model's text response.
2. Detect a `<think>...</think>` block if present.
3. Remove the entire reasoning block, including the tags.
4. Preserve the content after the closing `</think>` as the candidate final answer.
5. If there is no `<think>` block, preserve a normal final response unchanged except for safe whitespace normalization.
6. Trim leading/trailing whitespace.
7. If sanitization leaves an empty response, treat it as an invalid/empty model response and return the existing safe application error instead of displaying anything.

The implementation must handle the common case where the model response is exactly:

`<think> ... </think> FINAL SUMMARY`

and return only:

`FINAL SUMMARY`

Do not remove ordinary factual content merely because it contains normal Markdown or HTML-like text. The goal is specifically to remove the model reasoning wrapper/trace.

If the current Groq SDK/model provides a supported configuration that prevents reasoning output entirely, it may be used as an additional safeguard. However, **do not rely only on a model-specific behavior**. The application must still protect the user-facing output against an emitted `<think>` block.

## Prompt-Level Requirement

Also strengthen the existing server-side summarization instruction so the model is explicitly told:

- output only the final factual summary;
- do not output analysis or reasoning;
- do not output `<think>` tags;
- do not include self-review/checklists;
- do not describe its own generation process.

This is a secondary safeguard. Server-side sanitization remains mandatory.

## Factual Boundary — Preserve

Do not change the evidence boundary.

The model still receives only deterministic evidence such as:

- event type
- server timestamp
- latitude
- longitude
- GPS accuracy
- photo evidence presence

Do not add image analysis.

Do not add incident analysis.

Do not allow the model to infer causality, intent, blame, or unsupported facts.

## Error Handling

Preserve the existing safe error behavior.

If the model returns:

- empty text;
- only a reasoning block with no final answer;
- malformed/unusable output;
- provider failure;
- timeout;

the API should return a clear application-level error and the UI should show its existing safe error state.

Do not expose raw Groq responses, stack traces, API keys, or internal implementation details to the browser.

## Scope Discipline

Only make changes required for this output-polish fix.

Expected implementation surface is likely:

- `src/app/api/summary/route.ts`
- possibly the existing summary client component only if required for final rendering behavior

Do not change unrelated files.

Do not change the model-discovery architecture unless required to prevent reasoning output.

Do not add a new provider.

Do not add a database migration.

Do not change the event workflow.

## Verification Required

After implementation:

1. Run `npm run build` successfully.
2. Generate an AI Evidence Summary from the existing Timeline.
3. Confirm the browser displays **only the final factual summary**.
4. Confirm no `<think>` tag is visible.
5. Confirm no reasoning/self-review/checklist text is visible.
6. Confirm the summary still contains the correct Arrival → Check-in → Departure facts.
7. Confirm timestamps, locations, GPS accuracy, and photo-presence statements remain faithful to the deterministic evidence.
8. Confirm provider/model discovery still works.
9. Confirm an empty/invalid model response produces a safe error state.
10. Confirm Arrival, Check-in, Departure, and Timeline event cards remain unchanged.

If browser verification is performed by Antigravity, report the actual result. If it cannot be performed, report exactly what was verified and leave the manual verification requirement explicit.

## Required Report Update

Update:

`03_IMPLEMENTATION/implementation_reports/Chat6_Node3_Report_AIEvidenceSummaryImplementation.md`

Add a clearly marked section such as:

`## 15. Output Polish / Reasoning Trace Removal`

Include:

- root cause of visible `<think>` output;
- exact files changed;
- server-side sanitization behavior;
- prompt-level safeguard;
- empty-response handling;
- build result;
- browser/runtime verification result;
- whether the user-facing summary is now clean;
- any remaining limitation.

## Completion Boundary

This repair is complete only when the AI Evidence Summary remains functionally correct **and the Timeline UI displays only the final factual summary, with no visible reasoning trace**.

After successful verification, this polish/error-handling checkpoint can be marked **COMPLETED / LOCKED**.

**Do not expand scope beyond this output-polish repair.**
