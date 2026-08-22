# Chat6 Node3 Investigation: AI Evidence Summary Source Readiness

## 1. Investigation Objective
Perform a read-only source investigation of the current Freight application to determine implementation readiness of the next Core MVP feature: **AI Evidence Summary**.

## 2. Source Snapshot / Commit or Working-State Identifier
- **Environment**: Local working tree for Freight Hackathon
- **Current State**: Arrival, Check-in, Departure, and Timeline are fully implemented and verified. The Timeline correctly renders the factual chronological history of the trip.

## 3. Files Inspected
- `package.json`
- `.env.local`
- `src/app/api/` directory
- `src/app/(authenticated)/timeline/page.tsx`
- `src/app/(authenticated)/page.tsx`

## 4. Existing AI/LLM Infrastructure Findings
- **VERIFIED**: There are no existing AI SDKs, utility modules, prompt templates, or API routes in the codebase.
- **VERIFIED**: There are no AI-related environment variables (e.g., `OPENAI_API_KEY`) present in `.env.local`. 

## 5. Dependency / Package Findings
- **VERIFIED**: `package.json` contains no LLM dependencies. Implementing the single LLM call will require installing a new dependency (e.g., `@google/genai`, `openai`, or similar) or making a raw fetch request.

## 6. Deterministic Evidence Data Source Findings
- **VERIFIED**: The `events` data is securely accessed via Supabase SSR in `src/app/(authenticated)/timeline/page.tsx`.
- **VERIFIED**: The pattern securely resolves the driver (`auth_id`), fetches the active trip (`driver_id`), and queries the events (`trip_id`).
- **VERIFIED**: This exact data-access pattern can and should be reused. The query `events.select('*').eq('trip_id', trip.id).order('server_timestamp', { ascending: true })` provides all necessary deterministic evidence (types, locations, timestamps, photo existence).
- **VERIFIED**: The application can easily verify that all three events (Arrival, Check-in, Departure) exist by checking the length/types of the returned events array before passing data to the LLM.

## 7. AI Summary Trigger / Routing Findings
- **VERIFIED**: The current Dashboard (`page.tsx`) ends its state tracking at `hasDeparture` being true, presenting the "Trip Complete" state and a "View Timeline" CTA.
- **VERIFIED**: There is no existing placeholder or route for the AI Summary (e.g., `/ai-summary`). 
- **INFERRED**: The AI Summary CTA would naturally live either as a button at the bottom of the Timeline page (`/timeline`) or as a new state on the Dashboard after the trip is finalized. 

## 8. Supported Summary Output Requirements
- **VERIFIED**: The deterministic events provide: Event type, GPS latitude/longitude, server timestamp, and photo URL availability.
- **INFERRED**: The prompt passed to the LLM should strictly ask it to summarize the timeline based ONLY on this provided JSON array. It must be constrained by the prompt to avoid inventing intent or assigning blame.
- **UNKNOWN**: Specific layout constraints or exact wording requested by records are not explicitly defined beyond "factual summary".

## 9. LLM Input Boundary Findings
- **VERIFIED**: The database events array contains raw database fields (like `id`, `created_at`, `trip_id`, etc.) that are unnecessary for the LLM.
- **INFERRED**: The raw data should be transformed into a minimal structured JSON payload (e.g., `[{ type: "arrival", time: "...", lat: "...", lon: "...", photo_provided: true }]`) to save tokens and prevent confusion.
- **UNKNOWN**: It is currently unknown if the chosen LLM provider will be tasked with multimodal image analysis. Until instructed otherwise, `photo_url` should only be represented as boolean evidence (`photo_provided: true`).

## 10. Server-Side Security / API Boundary Findings
- **VERIFIED**: The LLM call must be performed securely on the server (e.g., in a Server Action or API route) so the API key is never exposed to the client.
- **VERIFIED**: The server route must implement the same trip-scoping authorization pattern used in `timeline/page.tsx` to prevent users from summarizing other drivers' trips.
- **VERIFIED**: The server must perform the evidence query itself rather than trusting a client-provided payload, ensuring that the AI input strictly reflects the immutable database state.

## 11. Error/Failure Readiness Findings
- **VERIFIED**: The app currently throws basic UI errors (e.g., 400, 401, 500) that can be extended for the AI route. 
- **INFERRED**: New error states will need to be handled, such as API timeouts or missing configurations.

## 12. VERIFIED / INFERRED / UNKNOWN Evidence Table

| Check / Requirement | Status | Evidence Source |
|---------------------|--------|-----------------|
| AI SDK installed | **VERIFIED** | `package.json` |
| AI API Keys in env | **VERIFIED** | `.env.local` |
| Existing AI API routes | **VERIFIED** | `src/app/api/` |
| Deterministic evidence available | **VERIFIED** | `timeline/page.tsx` data fetching |
| Three-event verification possible | **VERIFIED** | Events array length/types check |
| Target AI Provider/Model | **UNKNOWN** | Not defined in current source |
| Multimodal photo analysis required | **UNKNOWN** | Not defined in current source |

## 13. Blockers / Risks
- **Dependency Missing**: An AI provider SDK needs to be chosen and installed.
- **Configuration Missing**: The corresponding API key must be added to `.env.local`.
- **Security Boundary**: It is critical that the client does not send the event data to the server; the server must fetch it directly from Supabase to guarantee deterministic facts.

## 14. AI Evidence Summary Implementation Readiness Decision
**Pending Provider Decision.** The source data and architecture are fully ready to support the AI summary securely, but implementation cannot proceed until the specific AI provider/SDK is chosen and the API key is configured.

## 15. Smallest Safe Implementation Surface
Files to **ADD**:
- `src/app/api/summary/route.ts` (or Server Action)
- `src/components/AIEvidenceSummary.tsx` (Client component to trigger and display the summary)
Files to **CHANGE**:
- `src/app/(authenticated)/timeline/page.tsx` (To host the new AI Summary component)

## 16. Decisions Requiring ChatGPT/Ayush Approval
1. Which AI provider and model should be used? (e.g., OpenAI, Google Gemini).
2. Should the SDK be installed via npm?
3. Should the AI Summary CTA live on the Timeline page?

## 17. Out-of-Scope Findings
- None found during this inspection.

## 18. Recommended Next Step for ChatGPT/Ayush
Review this report, decide on the specific AI provider/SDK, and provide the API key. Once decided, issue the implementation instruction to Antigravity specifying the provider.
