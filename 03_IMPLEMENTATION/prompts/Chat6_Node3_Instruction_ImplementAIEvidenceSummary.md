# Freight — Chat6 Node3 Implementation Instruction: AI Evidence Summary

## Task

Implement **Core MVP Item #7 — AI Evidence Summary**.

The verified core workflow is:

**Arrival → Check-in → Departure → Trip Complete → Timeline → AI Evidence Summary**

Arrival, Check-in, Departure, and Timeline are already implemented and personally verified by Ayush. **Do not modify or re-investigate them unless a strictly necessary integration change is proven.**

## Source of Truth

Read and follow:

`03_IMPLEMENTATION/implementation_reports/Chat6_Node3_Report_AIEvidenceSummarySourceReadiness.md`

Also follow the locked Freight roadmap and existing project architecture.

## Locked AI Decision

The project will use:

- **AI provider:** Groq API
- **Credential:** `GROQ_API_KEY`
- The actual Groq API credential is stored privately in the environment by Ayush.
- **Do not hardcode or expose the key.**
- **Do not commit the secret to GitHub.**
- **Do not require Ayush to select or lock a specific model as a separate project decision.** Use the Groq API integration/configuration available to the project at implementation time.

The requirement is **free Groq API usage only**. Do not introduce a paid provider or paid service.

## Core Product Requirement

Generate **one factual AI evidence summary** from the deterministic stored trip evidence.

The AI must interpret/organize the evidence; it must NOT create evidence.

The AI must never invent:

- GPS coordinates
- timestamps
- events
- photo evidence
- driver intent
- blame
- unsupported conclusions

The database remains the source of truth.

## Required Data Flow

The server must perform the complete flow:

`Authenticated user → authenticated driver → relevant trip → events → validate Arrival + Check-in + Departure → build minimal deterministic evidence payload → one Groq LLM call → validate response → return summary`

Do **not** trust a client-provided `trip_id` or event payload as the source of truth.

The server must fetch the events directly from Supabase using the existing secure pattern already used by Timeline.

## Evidence Input

Use the existing `events` data and include only the minimum fields required for factual summarization.

At minimum, the deterministic payload should contain, when available:

- event type
- server timestamp
- latitude
- longitude
- GPS accuracy
- whether photo evidence exists

Do not send unnecessary database metadata such as internal IDs, auth IDs, or unrelated fields unless the implementation proves they are required.

### Photo Boundary

For this Core MVP, **do not implement multimodal photo analysis**.

Represent photo evidence as availability/presence only, for example:

`photo_provided: true/false`

The AI must not claim what is visible inside a photo because image analysis is not part of this task.

## Three-Event Gate

Before making the Groq call, the server must verify that the required event sequence exists:

- Arrival
- Check-in
- Departure

If one or more required events are missing, **do not call the LLM**. Return a clear application-level error explaining that the evidence summary requires the completed event sequence.

Do not fabricate a partial summary.

## Groq Integration

Use the existing `GROQ_API_KEY` environment variable server-side.

The implementation must:

- keep the API key server-only
- make the Groq request from the server
- use the Groq API correctly
- handle provider/API errors cleanly
- avoid logging the secret
- avoid returning the secret to the client

Do not add another AI provider.

Do not ask the user to provide the key through the browser.

If the installed/available Groq integration requires a model identifier at request time, use the appropriate currently available Groq configuration/model required by the API rather than turning that into a new product decision for Ayush. Do not hardcode a speculative/unsupported model ID.

## Prompt / AI Behavior

Construct a concise server-side instruction that tells the model:

1. It is summarizing Freight trip evidence.
2. The supplied structured data is the only source of truth.
3. It must summarize only facts present in that data.
4. It must not infer intent, blame, causality, or unsupported conclusions.
5. It must preserve the event sequence and recorded timestamps/locations accurately.
6. Missing photo evidence should be described as missing/not provided, not guessed.
7. It must not invent details.

Keep the prompt deterministic and focused.

## Output

Return a clean, user-readable factual summary.

The implementation may use structured output if the chosen Groq API configuration supports it, but do not introduce unnecessary complexity.

If structured output is used, validate the returned structure before displaying it.

If the model returns an invalid/empty response, show a clear error state rather than displaying fabricated or malformed content.

## Placement / UI

The source-readiness investigation identified two possible locations for the AI Summary CTA but did not lock the exact UI placement.

For this implementation, use the **Timeline page as the natural host** for the AI Evidence Summary because Timeline is the factual evidence view and the user reaches it after Trip Complete.

Make the smallest safe change necessary to provide:

- a clear **AI Evidence Summary** action on Timeline
- loading state while the server request is running
- summary display after success
- clear error state after failure

Do not redesign Timeline.

Keep the existing chronological event display intact.

## Expected Implementation Surface

The investigation proposed:

- `src/app/api/summary/route.ts` (or an equivalent server-side action if the existing architecture strongly favors it)
- `src/components/AIEvidenceSummary.tsx`
- minimal change to `src/app/(authenticated)/timeline/page.tsx` to host the feature

Use the smallest safe implementation surface.

If a different file structure is required by the actual current source, explain it in the implementation report.

## Security Requirements

The server must independently determine:

- authenticated user
- driver
- relevant trip
- event evidence

Never accept client-supplied evidence as authoritative.

Never expose `GROQ_API_KEY` to client-side JavaScript.

Never log the full API key.

Do not send another driver's events to Groq.

## Error Handling

Handle at minimum:

- unauthenticated request
- no relevant trip
- missing Arrival/Check-in/Departure
- missing `GROQ_API_KEY`
- Groq API/provider failure
- timeout/network failure
- empty model response
- malformed structured response if structured output is used

Return safe, user-readable errors without exposing secrets or internal credentials.

## Explicit Non-Goals

Do NOT:

- Modify Arrival.
- Modify Check-in.
- Modify Departure.
- Redesign Timeline.
- Change event schema.
- Add migrations.
- Change RLS policies.
- Implement AI inconsistency detection.
- Implement confidence/completeness scoring.
- Implement EXIF GPS/photo cross-checking.
- Implement natural-language Q&A.
- Implement multimodal photo analysis.
- Add another AI provider.
- Add paid AI services.
- Add multi-stop functionality.
- Fix unrelated architecture/security issues.

## Verification Required

Before reporting completion:

1. Run the project's appropriate type/lint/build checks.
2. Verify `npm run build` succeeds with no errors.
3. Verify the AI Summary action is accessible from Timeline.
4. Verify the server retrieves evidence from the authenticated user's trip rather than trusting client evidence.
5. Verify all three required events are checked before the LLM call.
6. Verify the Groq API call uses `GROQ_API_KEY` server-side.
7. Verify the API key is not exposed in client code or returned responses.
8. Verify a successful Groq response is displayed to the user.
9. Verify missing required events do not trigger an LLM call.
10. Verify missing API configuration produces a clear error rather than a crash.
11. Verify Groq/provider failure produces a safe error state.
12. Verify the AI summary does not invent evidence and is explicitly grounded in the deterministic payload.
13. Verify Timeline's existing chronological event display still works.
14. Confirm Arrival, Check-in, and Departure implementations were not unnecessarily changed.

If manual browser verification cannot be completed by Antigravity, report exactly what was verified and what remains for Ayush to verify. Do not claim manual verification that was not performed.

## Change Discipline

Keep changes minimal.

Before changing any file beyond the expected implementation surface, explain exactly why it is required in the report.

Do not change the database schema for this feature unless the actual source proves an unavoidable blocker; if such a blocker exists, stop and report it rather than silently changing architecture.

## Required Implementation Report

After implementation, create:

`03_IMPLEMENTATION/implementation_reports/Chat6_Node3_Report_AIEvidenceSummaryImplementation.md`

The report must include:

1. Summary of implementation.
2. Exact files added/changed.
3. Groq integration details (without exposing the secret).
4. How deterministic evidence is retrieved and scoped.
5. Three-event validation behavior.
6. AI prompt/input boundary.
7. Summary output behavior.
8. Security behavior.
9. Error handling.
10. Build/type/lint results.
11. Manual verification status.
12. Any deviations from this instruction.
13. Remaining work / Ayush manual verification steps.

## Completion Boundary

This task is complete when **AI Evidence Summary is implemented as a single server-side Groq LLM call over deterministic Arrival + Check-in + Departure evidence, the result is displayed from Timeline, automated checks pass, and no existing core event workflow is broken.**

Do not proceed to AI inconsistency detection or other stretch AI features after completing this task.

**Implement AI Evidence Summary only.**
