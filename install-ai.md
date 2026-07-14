# Hermes Brain — Installation Guide

## Quick Start

```bash
# 1. Clone the repo
git clone https://github.com/RohiRIK/hermes-brain.git ~/projects/hermes-brain
cd ~/projects/hermes-brain

# 2. Install Python dependency (only one: httpx for Gemini API)
pip install httpx

# 3. Set up Gemini API key (optional — for semantic vector search)
#    Get free key at: https://aistudio.google.com/apikey
export GEMINI_API_KEY="your-key-here"

# 4. Initialize the database (creates ~/.hermes/openltm.db)
python3 src/extraction/ltm_extraction_logic.py

# 5. Test recall
python3 -c "from src.embedding._db import MemoryDB; db = MemoryDB(); print(f'Memories: {db.count()}'); db.close()"
```

## What Gets Installed

| Component | Location | Purpose |
|---|---|---|
| `openltm.db` | `~/.hermes/openltm.db` | SQLite database (memories, FTS5, embeddings) |
| `httpx` | Python site-packages | HTTP client for Gemini API calls |
| `hermes-brain/` | `~/projects/hermes-brain/` | Source code, schema, migrations |

## Dependencies

### Required (Python stdlib only)
- `sqlite3` — database engine (built into Python)
- `re` — regex extraction (built-in)
- `struct` — binary vector packing (built-in)
- `logging`, `dataclasses`, `enum`, `typing` — all stdlib

### Optional (for Gemini embedding)
- `httpx` — async HTTP client for Gemini API
- `GEMINI_API_KEY` — Google AI API key (free tier: 1500 req/day)

## Database Schema

The DB is created automatically on first use. Schema source: `src/schema.sql`

### Tables
- `memories` — global learned insights (26 default entries)
- `context_items` — per-project goals, decisions, progress, gotchas
- `memory_files` — code anchors for staleness detection
- `memory_relations` — knowledge graph edges
- `memory_layout` — visual graph positions
- `tags` / `memory_tags` — many-to-many tagging
- `schema_migrations` — version tracking
- `settings` — key-value config store

### FTS5 Indexes
- `memories_fts` — full-text search on title + content
- `context_items_fts` — full-text search on context items

### Triggers
- `memories_ai/ad/au` — keep FTS in sync on insert/delete/update
- `context_items_ai/ad/au` — keep context FTS in sync

## Hermes Integration

The tools are built into Hermes Agent. No plugin installation needed.

### Available Tools
```python
openltm_recall(query="Docker networking", limit=5)    # Search memories
openltm_learn(content="Always use --compat", ...)     # Store new memory
openltm_forget(id=42, reason="Outdated")              # Delete memory
openltm_context(project="my-project")                  # Get project context
```

### Memory Categories
- `preference` — "I prefer X", "always use Y"
- `architecture` — "the system uses X", "we decided on Y"
- `gotcha` — "be careful", "watch out", "pitfall"
- `pattern` — "the pattern is", "this always happens"
- `workflow` — "the process is", "step 1, step 2"
- `constraint` — "must not", "required to", "limitation"

### Importance Levels
- `5` — never decays (critical constraints, core preferences)
- `4` — very slow decay (important architecture decisions)
- `3` — moderate decay (standard patterns)
- `2` — fast decay (minor preferences)
- `1` — test/ephemeral (cleaned up aggressively)

## Troubleshooting

### "no such table: memories"
Run the schema initialization:
```bash
sqlite3 ~/.hermes/openltm.db < src/schema.sql
```

### Gemini embedding not working
```bash
# Check API key is set
echo $GEMINI_API_KEY

# Test connection
python3 -c "
import httpx, os
r = httpx.get(f'https://generativelanguage.googleapis.com/v1beta/models?key={os.environ[\"GEMINI_API_KEY\"]}')
print(r.status_code, len(r.json().get('models', [])), 'models')
"
```

### FTS5 search returning nothing
Rebuild the FTS index:
```bash
sqlite3 ~/.hermes/openltm.db "INSERT INTO memories_fts(memories_fts) VALUES('rebuild')"
```

## File Structure

```
hermes-brain/
├── src/
│   ├── schema.sql                  # Canonical DB schema
│   ├── extraction/
│   │   └── ltm_extraction_logic.py # MemoryExtractor (845 lines)
│   └── embedding/
│       ├── __init__.py             # Public API
│       ├── _db.py                  # Hybrid FTS5 + vector search
│       └── _providers.py           # Gemini API client
├── migrations/                     # 23 versioned SQL migrations
├── scripts/                        # Utility scripts
├── AGENTS.md                       # Agent instructions
├── CLAUDE.md                       # Claude Code instructions
├── README.md                       # Project overview
└── install-ai.md                   # This file
```

## Next Steps

1. **Verify extraction works**: Run `python3 src/extraction/ltm_extraction_logic.py`
2. **Test recall**: Use `openltm_recall` in Hermes to search your memories
3. **Add Gemini embedding**: Set `GEMINI_API_KEY` for semantic search
4. **Customize categories**: Edit `CORRECTION_PATTERNS` in extraction logic
5. **Build graph relations**: Use `openltm_relate` to connect memories
