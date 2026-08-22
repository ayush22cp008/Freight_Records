# Freight — Chat6 Node3 Repair Instruction: Groq Model Discovery

## Purpose

Repair the existing **AI Evidence Summary** implementation.

The feature is already implemented. The current runtime failure is caused by an obsolete hardcoded Groq model:

`llama3-8b-8192`

Groq is returning a model-decommissioned error for that identifier.

**Do not rebuild the AI Evidence Summary feature. Fix only the Groq model-selection/configuration problem and verify the existing feature works.**

## Existing Implementation

Read first:

`03_IMPLEMENTATION/implementation_reports/Chat6_Node3_Report_AIEvidenceSummaryImplementation.md`

Also read:

`03_IMPLEMENTATION/prompts/Chat6_Node3_Instruction_ImplementAIEvidenceSummary.md`

The existing deterministic evidence pipeline, authentication, Supabase access, Timeline UI, and three-event gate are already implemented. Preserve them.

## Locked Product Decision

- Provider: **Groq API**
- Credential: **`GROQ_API_KEY`**
- The key is already stored in `.env.local` by Ayush.
- Keep the key server-side only.
- **Do not switch to another provider.**
- **Do not ask Ayush to manually select a model.**
- **Do not hardcode `llama3-8b-8192`.**
- **Do not hardcode any model that is not verified as currently active.**
- Cost requirement: **Free Groq usage only.**

## Critical Technical Fact

Groq's normal Chat/Responses API requires a `model` parameter. Therefore the application cannot literally make a model-less LLM request using only the API key.

However, the project requirement is that **Ayush should not manually choose or lock a model**.

Therefore implement **runtime model discovery** using Groq's Models API with the same `GROQ_API_KEY`:

`GET https://api.groq.com/openai/v1/models`

or the equivalent official Groq SDK method:

`groq.models.list()`

The application must use the returned active model information to determine a currently supported text-generation model for the summary request.

## Model Selection Rules

The application must select a model only after verifying that it is:

1. currently returned by Groq's active Models API;
2. suitable for text generation/chat completion;
3. available to the current project/key;
4. compatible with the existing request format;
5. within the project's **free-only** requirement.

Do not use an obsolete model.

Do not blindly choose the first item returned by `/models`.

Do not select audio-only, embedding-only, guard-only, or otherwise incompatible models for the evidence summary.

Do not select a paid-only model when the project is operating under the Free tier.

If the API response cannot establish a suitable free text-generation model, **do not guess**. Return a clear server error and report the exact blocker to Ayush.

## Important: No Manual Model Lock

Do NOT add a requirement such as:

`GROQ_MODEL=...`

that requires Ayush to manually select a model.

Do NOT tell Ayush to choose a model in the Groq dashboard.

The application should determine the currently usable model from Groq's available model information.

If a runtime environment variable is already present for a model, inspect it first, but do not require Ayush to add one merely to fix this task.

## Model Discovery Efficiency

Do not make an additional Models API request on every button click if the implementation can safely avoid that.

A simple server-side discovery/cache strategy is acceptable, provided that:

- the cache does not become a permanent stale model lock;
- a decommissioned-model error triggers rediscovery/retry once;
- the system does not loop indefinitely;
- the API key remains secret.

The actual AI summary should still remain **one LLM generation call** after model discovery. Model discovery is metadata/configuration retrieval, not an additional LLM call.

## Existing AI Evidence Flow — Preserve

Keep this exact logical flow:

`Authenticated user → authenticated driver → relevant trip → events → Arrival + Check-in + Departure validation → deterministic evidence payload → Groq model discovery → one Groq generation call → validate response → return summary`

Do not change:

- Arrival workflow
- Check-in workflow
- Departure workflow
- Timeline event rendering
- Supabase schema
- RLS policies
- deterministic evidence source
- three-event gate

## Evidence Boundary — Preserve

The AI input must remain only the deterministic stored evidence:

- event type
- server timestamp
- latitude
- longitude
- GPS accuracy, when available
- photo evidence presence, when available

Do not implement photo/image analysis.

Do not allow the model to invent evidence.

## Error Handling

Handle the existing decommissioned/invalid model case by rediscovering the currently active model and retrying the generation once.

Also handle:

- no active suitable model found
- Groq authentication failure
- Groq rate limit
- Groq provider failure
- network/timeout failure
- empty response

Return safe user-facing errors.

Never expose the API key or internal provider credentials.

Do not expose raw provider stack traces to the browser.

## Verification

After the fix:

1. Run `npm run build`.
2. Start the application.
3. Open the existing Timeline page.
4. Click **Generate AI Summary**.
5. Confirm the request no longer uses `llama3-8b-8192`.
6. Confirm a currently active Groq model is selected through discovery.
7. Confirm the Groq request succeeds.
8. Confirm the factual AI summary is displayed.
9. Confirm the summary is based only on Arrival + Check-in + Departure evidence.
10. Confirm the API key never appears in browser/client code or response payloads.
11. Confirm the existing Timeline event cards still work.
12. Confirm no unrelated files/features were changed.

If the Free-tier requirement cannot be verified programmatically, stop before making a paid-model request and report exactly what Groq exposes and what manual verification is required. **Never make a paid request merely to test the feature.**

## Required Report

After the repair, update/create:

`03_IMPLEMENTATION/implementation_reports/Chat6_Node3_Report_AIEvidenceSummaryImplementation.md`

Add a clearly marked **Repair / Model Discovery** section containing:

- root cause: obsolete `llama3-8b-8192` model;
- exact repair;
- how active model discovery works;
- how Free-tier eligibility is handled;
- selected model ID during verification (report it as a verification result, not as a new project lock);
- build result;
- runtime test result;
- whether AI summary was successfully generated;
- any remaining limitation.

## Completion Boundary

This repair is complete when the existing AI Evidence Summary works with the current Groq API without the obsolete hardcoded model, while keeping the project free-only and without requiring Ayush to manually select a model.

**Do not expand scope beyond this Groq model-discovery/runtime repair.**
