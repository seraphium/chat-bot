# Enhancement Plan — Chat-Bot Repository

This plan prioritizes missing modern chatbot capabilities and outlines phased, actionable improvements across backend (FastAPI/LangChain) and frontend (Next.js). The goal is safer reasoning, better reliability, richer retrieval, and improved UX without breaking current APIs.

## Summary of Gaps
- Realtime UX: SSE-only; no WebSocket channel for typing/presence or multi-client sync.
- Conversation storage: Messages stored as JSON on conversation; no message-level rows, edits, reactions, or analytics.
- Safety: No input/output moderation, PII redaction, or rate limiting on chat endpoints.
- Observability: No Prometheus metrics or tracing; limited error visibility.
- Tool calling: No structured tool/function use for search/calculator/file ops.
- RAG: Basic splitting; limited file loaders; no citations/re-ranking.
- Multimodal: Image path tied to Ollama; no OpenAI Vision; no audio/voice.
- Frontend: Missing citations UI, message edit/regenerate, export/share, per-conversation settings.

## Phase P0 — Core Reliability and Safety

1) Safe Reasoning Mode (Chain-of-Thought) — First Feature
- Approach: Enable internal reasoning traces while not exposing full chain-of-thought to users.
- API: Extend chat request with `reasoning_mode: "off" | "hidden" | "concise"` and optional `reasoning_effort: "low" | "medium" | "high"`.
- Backend: For OpenAI reasoning-capable models, pass reasoning config; for others, instruct “think step-by-step internally, output final answer only.” Never persist or stream raw CoT. If `concise`, stream a short, redacted rationale (SSE event type `rationale`) capped by length and filtered for PII.
- UI: Toggle “Reason better” and optional “Show brief rationale (beta)”; defaults to off. Display brief rationale in a collapsible panel; never show raw thoughts.
- Metrics: Track reasoning usage, token deltas, and error rate.

2) Message Model + Migrations
- Add `messages` table (id, conversation_id, role, content, metadata JSON, tokens, cost, created_at). Update ChatService to write per message and deprecate the JSON blob field progressively.

3) Rate Limiting and Moderation
- Apply Redis-backed rate limits (e.g., fastapi-limiter) on `/auth/*` and `/chat/*` with `429` + `Retry-After`.
- Add optional moderation: lightweight input filter (PII/keys) and provider moderation for outputs. Redact or block with actionable errors. Feature-flagged.

4) Observability
- Expose `/metrics` (Prometheus) with request/stream counters and latency histograms. Add structured logging and basic tracing (OTel) for FastAPI + httpx.

5) Realtime Channel
- Add `/ws` for typing indicators, presence, and live status; keep content over SSE for stability. Heartbeats + auto-reconnect.

Acceptance (P0)
- Reasoning: Final answers never include raw CoT; optional concise rationale gated and length-limited.
- DB: Messages persisted per-row; reads still work for existing JSON until removed.
- Limits: `/chat/*` enforces rate limits with tests and backoff guidance.
- Metrics: `/metrics` shows p95 latency, error rate, stream counts.
- Realtime: Typing/presence visible across clients via WebSocket.

## Phase P1 — Intelligence, Retrieval, and UX

1) Tool/Function Calling
- Implement tool schemas for web_search, file_lookup, and calculator. Support OpenAI tool-calling; provide a LangGraph path for local/Ollama. Route based on provider.

2) RAG Upgrades
- Switch to `RecursiveCharacterTextSplitter`; use `unstructured` loaders (DOCX/PPTX/HTML/TXT). Embed metadata (source, page, chunk_id). Stream citations as side-band events; optional re-ranking.

3) Frontend Enhancements
- Citations panel with source drill-down; message edit + regenerate; conversation export (Markdown/JSON); per-conversation settings (model/provider, context K, web-search toggle).

4) Vision Support
- Add OpenAI Vision path; detect images in conversation and route accordingly.

Acceptance (P1)
- Tool calling executes web search or file lookup and streams citations to UI.
- DOCX/PPTX/TXT/HTML processed; citations show chunk-level source.
- Users can edit/regenerate messages and export conversations.

## Phase P2 — Advanced/Multi-tenant

1) Per-User Provider Keys
- Secure storage, policy controls (allowed models, rate-limits), and UI for key entry.

2) Content Scanning
- Antivirus (ClamAV) + strict MIME validation for uploads; quarantine on fail.

3) Retrieval Quality
- Hybrid search (BM25 + vector), re-ranker toggle, dedupe/score normalization.

4) Voice
- Optional ASR/TTS (Whisper + ElevenLabs or alternatives) with streaming where possible.

Acceptance (P2)
- User-scoped keys with policy enforcement and audit logging.
- Blocked malicious files produce clear, user-facing errors.

## API and Schema Deltas (Focused)
- `POST /api/v1/chat/conversations/{id}/messages` request: add `reasoning_mode`, `reasoning_effort` (optional). Default `off`.
- SSE: optional `rationale` event when `reasoning_mode="concise"`; `content` and `complete` semantics unchanged.
- DB: Alembic migration adds `messages` table; conversations JSON field marked deprecated.

## Initial Workstream (Recommended PR Order)
1) P0 “Reasoning + Limits + Metrics + Messages”
   - Alembic migration + repository for messages.
   - ChatService updated to persist messages and support `reasoning_mode` without exposing raw CoT.
   - Rate-limiter middleware integration and tests.
   - `/metrics` with counters/histograms; structured logging.
2) WebSocket presence/typing channel + UI toggles (reasoning + brief rationale).
3) P1 tool calling and RAG upgrades; citations UI.

Notes
- Reasoning safety: Never log or return full chain-of-thought. Store only minimal metadata (e.g., token counts, effort) and optionally a short redacted rationale when explicitly enabled.
- Backwards compatibility: Maintain existing SSE shape; add events only when features enabled.

