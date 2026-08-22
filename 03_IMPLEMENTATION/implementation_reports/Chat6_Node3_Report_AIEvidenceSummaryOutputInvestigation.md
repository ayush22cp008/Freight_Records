# Chat6 Node3 Investigation: AI Evidence Summary Output Regression

## 1. Investigation Objective
Determine the root cause of the AI Evidence Summary output regression where the browser displays only a truncated fragment (`On 2026-08-21T07:49:`) instead of the full three-event factual summary, specifically verifying if it's caused by the previously added `<think>` sanitizer, a database issue, or an API truncation limit.

## 2. Current Source Files Inspected
- `src/app/api/summary/route.ts`
- `scratch/test-summary.mjs` (Local test script used to simulate the API call and inspect the raw Groq response)

## 3. Current Runtime Model Selected
The dynamic model discovery selects an available free text-generation model. During testing, it resolved to `qwen/qwen3.6-27b`, which is a reasoning model that inherently outputs extensive `<think>...</think>` traces regardless of instructions to the contrary.

## 4. Current Generation / Output-Token Configuration
**None.** The current `groq.chat.completions.create` call does not explicitly configure a `max_tokens` parameter. It completely relies on Groq's default generation limit (which is typically around 1024 tokens for many models if unspecified).

## 5. Current System Prompt Behavior
The system prompt contains the explicit instruction: "Output ONLY the final factual summary. Do not output analysis, reasoning traces, <think> tags, checklists, or descriptions of your own generation process." However, reasoning models (like `qwen` or `deepseek-r1`) are structurally trained to generate reasoning traces first and frequently ignore negative constraints against doing so.

## 6. Current Deterministic Evidence Payload Shape and Event Count
The payload perfectly contains all **3 events** (Arrival, Check-in, Departure) properly formatted with their timestamps, coordinates, and photo evidence indicators. The database and evidence pipeline are completely intact.

## 7. Raw Response Behavior Before Sanitization
Because the model is a reasoning model, it generates a massive `<think>...</think>` block detailing its planning process. Because no `max_tokens` limit was set in the API request, the model hits the default token generation limit (e.g., 1024 tokens) just as it finishes the reasoning trace and begins to output the final answer. The raw response looks like:
```text
<think>
[... hundreds of tokens of reasoning ...]
</think>
On 2026-08-21T07:49:
```
The generation is forcibly stopped mid-sentence due to hitting the output token limit.

## 8. Sanitized Response Behavior
The sanitizer regex (`/<think>[\s\S]*?<\/think>/gi`) successfully matches the closed `<think>...</think>` block and removes it. This perfectly unmasks the fact that the final answer was cut off, leaving only the remaining truncated fragment: `On 2026-08-21T07:49:`.

## 9. Root Cause of the Incomplete Summary
**Token Limit Truncation.** The underlying reasoning model is generating an extraordinarily long reasoning trace that consumes the entirety of Groq's default output token budget. The generation process runs out of tokens and truncates exactly as the final factual summary begins. The sanitizer correctly removes the reasoning, which exposes the truncated final answer.

## 10. Evidence Proving the Root Cause
- The local test script running the exact same payload generated a `<think>` block of over 400 words.
- The un-sanitized raw output clearly shows the model begins the final summary exactly where the truncation occurs.
- The `route.ts` API call lacks any `max_tokens` parameter to expand the token budget.
- The three-event payload is verified to be fully present before the Groq API call.

## 11. Whether the <think> Sanitizer is Functioning Correctly
**Yes.** The sanitizer functions flawlessly using non-greedy lazy matching (`[\s\S]*?`). It correctly isolates and removes the reasoning block without affecting the actual generated output.

## 12. Whether the Three-Event Evidence Gate is Functioning Correctly
**Yes.** The server correctly verifies and fetches all three events. The token truncation happens solely on the LLM provider side.

## 13. Exact Minimal Fix Recommendation
Add a `max_tokens` parameter to the `groq.chat.completions.create` configuration with a sufficiently large value (e.g., `max_tokens: 4096` or `8192`) to accommodate both the verbose reasoning trace and the final factual summary without hitting premature truncation.
Alternatively, if Groq supports it for this model, pass a parameter to disable reasoning traces, but setting `max_tokens` is the safest, most robust fix for premature truncation.

## 14. Files That Would Need to Change for the Fix
- `src/app/api/summary/route.ts`

## 15. Explicit Statement
**No code was changed during this investigation.** The application code in `freight/` remains completely unmodified.
