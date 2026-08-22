# Freight — Chat6 Node3 Instruction: Fix AI Evidence Summary Truncation

## Objective

Fix the AI Evidence Summary regression identified in:

`03_IMPLEMENTATION/implementation_reports/Chat6_Node3_Report_AIEvidenceSummaryOutputInvestigation.md`

The current browser output can collapse to a fragment such as:

`On 2026-08-21T07:49:`

because the dynamically selected reasoning model spends the output budget generating reasoning before the final factual summary.

The fix MUST remain on the **Groq Free tier**. Do not introduce OpenAI, Anthropic, paid Groq Developer-tier requirements, or another provider.

## Important Product Requirement

The user-facing AI Evidence Summary must contain a complete, concise factual summary of ALL deterministic events:

1. Arrival
2. Check-in
3. Departure

The output must not expose model reasoning or `<think>` traces.

## Existing Architecture to Preserve

Do NOT change:

- Arrival implementation
- Check-in implementation
- Departure implementation
- Timeline implementation
- database schema
- RLS policies
- authentication architecture
- deterministic evidence collection
- three-event evidence gate
- Groq API key architecture
- `.env.local` secret handling

The existing server-side evidence pipeline is correct and must remain the source of truth.

## Free Groq Model Requirement

Use only a model available through the Groq API under the Free tier limits.

Do NOT require the user to purchase a Groq Developer plan or add billing.

Groq's current documentation lists `qwen/qwen3.6-27b` among models with Free Plan rate limits and documents that Qwen 3.6 supports `reasoning_effort: "none"`, which disables reasoning tokens. Prefer this configuration when that model is available.

If the existing dynamic model-discovery architecture is retained, make the selection **capability-aware** rather than blindly selecting the first model:

1. Prefer a currently available Free-tier text model that supports disabling reasoning.
2. For `qwen/qwen3.6-27b`, use:
   `reasoning_effort: "none"`
3. If the selected Free-tier model is GPT-OSS and it supports the relevant API option, use `include_reasoning: false` so reasoning is not returned to the user.
4. Do not select a reasoning configuration that deliberately generates raw `<think>` output for this feature.
5. If no compatible Free-tier text model is available, fail clearly with a user-safe error rather than silently selecting a paid model.

Do NOT hardcode a paid model.

Do NOT require the user to manually select a model in the Groq console. The application should continue using the existing `GROQ_API_KEY` and server-side model selection logic.

## Generation Configuration

Set an explicit reasonable output limit so the factual summary cannot be truncated merely because of provider defaults.

Use a conservative value appropriate for this short summary (for example 1024–2048 output tokens), not an unnecessarily huge value such as 8192.

The goal is NOT to give a reasoning model more room to think. The goal is to disable unnecessary reasoning and reserve a bounded amount of output for the actual summary.

Use the exact parameter supported by the current Groq SDK/API version in the repository. Do not invent parameter names.

## Prompt Requirements

Keep the existing factual constraints, including:

- summarize ONLY the supplied deterministic evidence;
- do not infer causes, blame, intent, or facts not present in the payload;
- preserve event sequence;
- preserve timestamps;
- preserve coordinates and GPS accuracy when provided;
- accurately state whether photo evidence was provided;
- do not invent details;
- return only the final user-facing summary.

The model should produce a concise final answer covering all three events.

## Sanitizer

KEEP the existing `<think>` sanitizer as a defensive safety layer.

Do not remove it merely because reasoning is now disabled.

The desired path is:

Deterministic evidence
→ Free Groq text model with reasoning disabled/hidden
→ concise factual answer
→ `<think>` sanitizer safety layer
→ UI

## Required Validation

After implementation, test the actual browser flow using the existing trip containing:

- Arrival
- Check-in
- Departure

Verify that the returned summary contains all three event types.

The final UI must NOT show:

- `<think>`
- `</think>`
- model reasoning
- "analysis"
- "self-correction"
- "final check"
- generation-process narration

The final UI SHOULD contain a concise factual summary similar in substance to:

- Arrival recorded at its timestamp/location, with photo evidence status.
- Check-in recorded at its timestamp/location, with photo evidence status.
- Departure recorded at its timestamp/location, with photo evidence status.

Do not hardcode this example text. It must be generated from the actual deterministic evidence payload.

## Regression Checks

Verify:

1. All three events are present in the server payload immediately before the Groq call.
2. The selected model is Free-tier compatible.
3. Reasoning is disabled or excluded from the user-facing output.
4. The response is not truncated.
5. `<think>` sanitization still works as a fallback.
6. Errors from Groq are handled without exposing secrets.
7. `GROQ_API_KEY` is never logged or rendered.
8. Existing Arrival/Check-in/Departure/Timeline behavior is unchanged.
9. The project builds successfully.
10. The AI summary can be regenerated successfully.

## If Dynamic Model Discovery Is Changed

Do not remove dynamic discovery without a concrete reason.

If changes are required, document exactly:

- how available models are discovered;
- how Free-tier-compatible models are identified/preferred;
- how reasoning capability is detected/handled;
- what fallback occurs when the preferred model is unavailable.

Do not claim a model is Free based only on its name. Verify against the current Groq model/rate-limit information available to the application or repository documentation.

## Scope

Primary implementation file expected:

`src/app/api/summary/route.ts`

Only modify additional files if strictly required for the fix.

## Required Implementation Report

Create/update:

`03_IMPLEMENTATION/implementation_reports/Chat6_Node3_Report_AIEvidenceSummaryFix.md`

Include:

1. Root cause from the investigation report.
2. Exact implementation change.
3. Exact model-selection behavior.
4. Confirmation that only Free-tier Groq usage is required.
5. Reasoning configuration used.
6. Output-token configuration used.
7. Sanitizer status.
8. Three-event evidence-gate status.
9. Browser test result showing Arrival + Check-in + Departure in the final summary.
10. Confirmation that no API key or secret was exposed.
11. Build/test result.
12. Files changed.

## Completion Condition

Do not mark the feature complete unless the browser visibly shows a **complete three-event AI Evidence Summary** and no reasoning trace.

The final expected behavior is:

**Arrival → Check-in → Departure → complete concise factual AI summary**

using the existing Groq API key and **Free-tier Groq usage only**.
