# hermes-brain

Long-term memory system for Hermes Agent. Derived from OpenLTM's schema with Hermes-native Python integration.

## Working in this repo

- **Python** — extraction logic and embedding module. Use `python3`, venv optional.
- **SQLite** — schema + migrations. DB file lives in `~/.hermes/openltm.db` (not in repo).
- **Tests** — run `python3 src/extraction/ltm_extraction_logic.py` for extraction demo.

## Key files

- `src/schema.sql` — canonical DB schema
- `src/extraction/ltm_extraction_logic.py` — MemoryExtractor (deterministic, no LLM)
- `src/embedding/` — Gemini API + hybrid FTS5/vector search
- `migrations/` — versioned SQL migrations

## Rules

- **Never commit `.db` files** — personal memory data stays local
- **Schema changes** go in `src/schema.sql` AND matching migration file
- **Free-tier only** — no paid model APIs for embedding (Gemini free tier)
