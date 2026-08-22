# DECISIONS.md

## Core MVP — AI Evidence Summary / Groq

**Decision date:** Aug 22, 2026

- Use Groq for the Core MVP AI Evidence Summary.
- `GROQ_API_KEY` remains strictly server-side and is read from the server environment.
- The application dynamically discovers/selects an available text-generation model through the active Groq API key rather than hardcoding a single project-selected model.
- The implementation is constrained to models available to the active/free-tier API access; no paid model is intentionally requested.
- The AI receives deterministic Arrival, Check-in, and Departure evidence only.
- AI produces a concise factual summary; it must not generate or replace GPS, timestamps, event types, or stored evidence.
- The server-side `<think>` sanitizer remains a defensive layer so reasoning traces are not exposed in the UI.
- The AI Summary truncation fix uses an explicit output-token budget and disables unnecessary Qwen reasoning where applicable.

## Existing locked decisions

Product definition, Core MVP scope, stack, event schema, RLS/immutability architecture, and core navigation remain locked. Changes to those areas require an explicit new decision and reason.
