# Hermes Brain

Long-term memory system for Hermes Agent — the "brain" behind persistent knowledge across sessions.

Built on [OpenLTM](https://github.com/RohiRIK/OpenLtm)'s schema and SQLite engine, with Hermes-native integration via Python tools instead of MCP.

## What is this?

Hermes Agent ships with `openltm_recall`, `openltm_learn`, `openltm_forget`, and `openltm_context` tools. This repo contains:

- **Schema** — SQLite tables for memories, context items, embeddings, relations, graph layout
- **Extraction** — Deterministic `MemoryExtractor` with precompiled regex and 3-layer gating (no LLM calls)
- **Embedding** — Gemini API integration for semantic vector search (lazy single-embed on learn)
- **Migrations** — 23 schema migrations from OpenLTM

## Architecture

```
┌──────────────────┐     ┌──────────────────┐     ┌─────────────────┐
│  MemoryExtractor │────▶│   SQLite DB      │────▶│  Hermes Tools   │
│  (extraction/)   │     │  (openltm.db)    │     │  recall/learn/  │
│  regex + scoring │     │  FTS5 + vectors  │     │  forget/context │
└──────────────────┘     └──────────────────┘     └─────────────────┘
        │                         │
        ▼                         ▼
┌──────────────────┐     ┌──────────────────┐
│  Gemini Embed    │     │  Decay Engine    │
│  (embedding/)    │     │  importance ×    │
│  cosine search   │     │  usage recency   │
└──────────────────┘     └──────────────────┘
```

## Files

```
src/
├── schema.sql              # Full DB schema (memories, context, FTS5, relations)
├── extraction/
│   └── ltm_extraction_logic.py  # MemoryExtractor — deterministic fact extraction
└── embedding/
    ├── __init__.py         # Public API: MemoryDB, GeminiProvider
    ├── _db.py              # SQLite store with hybrid FTS5 + vector search
    └── _providers.py       # Gemini embedding API client

migrations/                 # 23 versioned SQL migrations (001–023)
scripts/                    # Utility scripts
```

## Usage (in Hermes)

The tools are built into Hermes — no installation needed:

```python
# Search memories
openltm_recall(query="Docker networking gotchas", limit=5)

# Store a new memory
openltm_learn(content="Always use --compat on docker swarm init", category="gotcha", importance=4)

# Get project context
openltm_context(project="hermes-brain")

# Delete outdated memory
openltm_forget(id=42, reason="Superseded by new approach")
```

## Extraction Logic

`MemoryExtractor` processes conversations turn-by-turn:

1. **Signal detection** — regex patterns identify correction, preference, architecture, gotcha moments
2. **Noise filtering** — trivial/obvious statements are rejected
3. **Scoring** — importance (1–5) and confidence (0.0–1.0) with thresholds
4. **Rate limiting** — max 2 facts per turn, 5 per session

No LLM calls. Pure heuristics. O(n) scan with precompiled patterns.

## Gemini Embedding

Hybrid search flow:
1. FTS5 BM25 → candidate IDs
2. If candidates < limit → embed query → cosine similarity → augment
3. Merge, dedupe, sort by (semantic_score, fts_rank, decay_score)

Embedding is computed lazily on `learn()` — single embed per memory, no batch backfill needed for <10K memories.

## License

MIT — same as OpenLTM.
