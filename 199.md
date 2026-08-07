# Agentic RAG HTTP MCP Server — Spec

## Problem Statement

Jyasu wants his coding agents (Claude Code, Claude Desktop, and other MCP-capable
clients) to be able to retrieve relevant context from an existing knowledge corpus
— stored in Postgres — during agentic sessions, over HTTP. The corpus is primarily
Chinese-language content, mixed with English/code identifiers (function names,
error codes), so retrieval needs to handle both natural-language semantic queries
and exact-term lookups well. Today there's no MCP-accessible retrieval layer at
all; agents have no way to pull in this context without it being manually pasted
into the conversation.

## Solution

A thin, stateless Rust MCP server exposed over streamable HTTP (`rmcp` + `axum`)
that gives any MCP client a small set of retrieval primitives. The server itself
does no multi-step reasoning — it runs a layered retrieval funnel (keyword
pre-filter → conditional ANN vector search → weighted scoring) per call and
returns ranked, snippet-level results. The calling agent decides when to search,
how to refine queries, and when to fetch full document content — consistent with
Jyasu's existing `FieldRule`-style layered-funnel architecture from the
duplicate-detection system.

## User Stories

1. As an agent working in Claude Code, I want to search the knowledge base with a
   natural-language query, so that I can retrieve relevant context without the
   user manually providing it.
2. As an agent, I want to search using an exact term (function name, error code,
   Chinese keyword), so that I get a precise match rather than noisy semantic
   neighbors.
3. As an agent, I want to explicitly request keyword-only or semantic-only search
   via a `mode` parameter, so that I can apply my own judgment about the query
   shape instead of trusting a hidden default.
4. As an agent, I want `search` to default to a smart hybrid behavior when I don't
   specify a mode, so that I get good results without needing to understand the
   server's internal funnel.
5. As an agent, I want search results returned as short snippets rather than full
   document content, so that I don't blow my context window on irrelevant hits.
6. As an agent, I want to fetch the full content of a specific chunk/document by
   ID after reviewing snippets, so that I can get complete context only for
   results I've judged relevant.
7. As an agent, I want snippets from keyword-matched results to highlight the
   matching terms in context, so that I can judge relevance faster than reading a
   blind truncation.
8. As an agent, I want to call `keyword_search` directly when I already know I
   need an exact-term lookup, so that I can skip the ANN stage entirely and get a
   faster, more precise response.
9. As an agent, I want to call `vector_search` directly when I have a vague,
   intent-based query, so that I can skip keyword pre-filtering when I know it
   won't help.
10. As Jyasu (operator), I want the server to authenticate requests via a bearer
    token, so that it isn't open to unauthenticated access when deployed off
    localhost.
11. As Jyasu (operator), I want Chinese-language queries to be handled with
    proper word segmentation (via Elasticsearch + ik_analyzer), so that keyword
    search precision doesn't degrade the way naive tsvector/trigram matching
    would for CJK text.
12. As Jyasu (operator), I want English/code content to be searchable via
    Postgres tsvector, so that identifier-style queries (function names, error
    codes) get exact, low-latency matches without going through Elasticsearch.
13. As Jyasu (operator), I want a pg_trgm fallback path available, so that
    keyword search still functions in environments where Elasticsearch isn't
    running or hasn't been synced yet.
14. As Jyasu (operator), I want Postgres → Elasticsearch sync handled via
    CDC (reusing `pg_x`-style logical replication), so that ES stays consistent
    with Postgres without dual-write inconsistency risk.
15. As Jyasu (operator), I want query embeddings generated locally (BGE-M3 via
    `ort`), so that the server has no external embedding API dependency, cost, or
    latency for the ANN stage.
16. As Jyasu (operator), I want the ANN stage to be skipped automatically when
    keyword pre-filtering already returns confident results, so that the common
    case (exact/structured queries) stays fast and cheap.
17. As Jyasu (operator), I want the weighted scoring stage to combine exact-match
    score, ANN similarity, and metadata signals via fixed coefficients, so that
    ranking is deterministic and reuses the proven pattern from the
    duplicate-detection system.
18. As a future maintainer, I want the funnel logic (`RetrievalFunnel`) decoupled
    from the MCP transport layer, so that it can be tested against a real
    Postgres+ES instance without spinning up the HTTP/MCP protocol stack.

## Implementation Decisions

- **Transport / protocol**: `rmcp` (official Rust MCP SDK) mounted as a Tower
  service inside an `axum::Router`, using streamable HTTP transport. Per the SDK's
  default model, the service factory runs per request (stateless); shared state
  (DB pool, ES client, embedding model handle) is held in a `Clone`-able struct
  captured by the factory closure, not in-memory session state.
- **Tools exposed**:
  - `search` — primary tool. Runs the full funnel. Accepts `query`, optional
    `mode` (`"keyword" | "semantic" | "hybrid"`, default `"hybrid"`), optional
    filters (source, language, date range). Returns ranked results as snippets.
  - `keyword_search` — runs only the pre-filter stage (ES / tsvector / pg_trgm
    depending on config/content type).
  - `vector_search` — runs only the ANN stage against pgvector.
  - `fetch_by_id` — returns full content for a given chunk/document ID.
- **Funnel stages** (mirrors the `FieldRule` enum pattern from the
  duplicate-detection system):
  1. **Pre-filter (exact/keyword)** — pluggable strategy per content type:
     - Elasticsearch + `ik_analyzer` — primary strategy for Chinese content.
     - Postgres `tsvector` — for English/code content (identifiers, error codes).
     - `pg_trgm` — fallback when ES is unavailable/unsynced.
  2. **ANN (conditional)** — pgvector cosine/L2 search. Only runs when: (a) agent
     explicitly requests `mode: "semantic"` or `"hybrid"`, or (b) auto
     short-circuit logic determines the pre-filter stage didn't return enough
     high-confidence hits. Query embedding produced locally via BGE-M3 (`ort`
     runtime).
  3. **Scoring** — fixed-coefficient weighted combination of exact-match score,
     ANN similarity, and metadata signals (freshness, source weight), matching
     the dedup system's `ScoringConfig`-style approach. Coefficients live in a
     single config struct, not scattered through funnel logic.
- **Sync**: Postgres → Elasticsearch via CDC, reusing `pg_x`-style logical
  replication patterns. A consumer service applies Postgres changes (insert/
  update/delete) to the ES index.
- **Result payload / progressive disclosure**: `search` and `keyword_search`
  return snippets, not full chunk content — id, source, score, and a short
  context window. ES-matched results use ES highlight for the snippet; ANN-only
  matches fall back to fixed truncation. Full content requires a follow-up
  `fetch_by_id` call.
- **Auth**: bearer token checked via axum middleware, sourced from config/env.
  Upgradeable to OAuth (already supported by `rmcp`) without touching tool logic,
  if the server is ever exposed to external/multi-tenant clients.
- **Observability**: `tracing` + structured logs only for v1. Full OTel
  (traces/metrics wired to existing Jaeger/Prometheus stack) deferred until
  funnel/scoring behavior is stable enough to know which spans/metrics matter.
- **Embedding**: BGE-M3, run in-process via `ort` (ONNX Runtime) — chosen for
  strong native Chinese support and to avoid an external embedding API
  dependency. No existing embedder to inherit from; this is a new choice, not a
  constraint from prior infrastructure.

## Testing Decisions

- **Primary seam**: `RetrievalFunnel` — a transport-independent service that
  takes a query + `FieldRule`-style stage config and returns scored results.
  Postgres and ES clients are injected as trait objects, so tests can run
  against real instances (e.g. via Testcontainers) rather than mocks, keeping
  tests close to production behavior without spinning up the MCP/HTTP layer.
- **What "good" looks like**: tests exercise `RetrievalFunnel::search` with real
  fixture data across representative query shapes (exact Chinese term, exact
  English identifier, vague Chinese query, vague English query) and assert on
  returned ranking/short-circuit behavior — not on internal stage call counts or
  implementation details.
- **Modules tested**:
  - `RetrievalFunnel` (integration-style, against real Postgres+ES via
    Testcontainers) — the main test surface.
  - Individual `FieldRule` strategy implementations (tsvector, pg_trgm, ES) —
    unit-level, each against a minimal fixture dataset, to isolate strategy-
    specific bugs from funnel orchestration bugs.
  - Scoring stage — pure unit tests, given fixed inputs, since it's
    deterministic arithmetic with no I/O.
  - MCP tool handlers (`search`, `keyword_search`, `vector_search`,
    `fetch_by_id`) — thin, so tested only for correct parameter mapping into
    `RetrievalFunnel` calls and correct MCP response shaping, not funnel logic.
- **Prior art**: Jyasu's dedup system and `pg_x` CDC work presumably have
  existing integration test patterns against real Postgres instances — reuse
  those Testcontainers/fixture conventions rather than introducing a new
  testing approach for this project.

## Out of Scope

- Ingestion pipeline for populating Postgres/pgvector — this spec assumes
  content and embeddings are already being written to Postgres by an existing or
  separate process.
- Cross-encoder / LLM-based reranking.
- Tunable/per-query scoring coefficient overrides.
- Semantic highlighting for ANN-only hits (fixed truncation fallback only, v1).
- pg_trgm fallback activation thresholds and monitoring.
- `candle`-based native embedding path (ORT is the v1 choice).
- Full OTel instrumentation.
- OAuth-based auth.
- Any UI, dashboard, or non-MCP client integration.

## Further Notes

- This spec assumes a corpus that is majority Chinese-language with a meaningful
  minority of English/code content (identifiers, error codes) — the dual
  tsvector/ES split and BGE-M3's multilingual strength are both direct responses
  to that mix, not a generic default.
- The auto short-circuit threshold (what counts as "confident enough" to skip
  ANN) starts as a rough constant and is expected to need calibration once real
  query traffic is observed — tracked in the opportunity list, not blocking v1.
- Opportunity list (deferred, not blocking, tracked for future iteration):
  cross-encoder/LLM rerank, tunable scoring weights, ANN-only semantic
  highlighting, pg_trgm activation triggers, `candle` embedding alternative,
  short-circuit threshold tuning, full OTel, OAuth upgrade.
