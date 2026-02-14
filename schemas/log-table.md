# The Log Table — Unified Append-Only Primitive

## What It Is

`reeves.log` is the single most important table in Reeves. It replaced 14 separate append-only tables with one unified primitive. Every event, transcript, devlog, completion, note, and observation flows through the log.

**Bright-line rule:** Immutable + AI context → Log.

**Column rule:** Filter by it → column. Read it → metadata/content.

## Schema

```sql
CREATE TABLE reeves.log (
    id              BIGSERIAL PRIMARY KEY,
    ts              TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    -- Filter predicates (columns you filter by, not data you read)
    source          TEXT NOT NULL,        -- origin: 'chat_log', 'devlog', 'task_completion', etc.
    entity_type     TEXT,                 -- parent: 'task', 'skillflow', 'conversation'
    entity_id       TEXT,                 -- FK-by-convention to any entity
    event_type      TEXT,                 -- action: 'completed', 'noted', 'voice.session_start'
    direction       TEXT,                 -- flow: 'in', 'out', 'internal'
    actor           TEXT,                 -- who: 'claude-code', 'user', 'reeves-brain'
    tags            TEXT[] DEFAULT '{}',  -- flexible categorization

    -- Payload (data AI reads via search)
    content         TEXT,                 -- the actual text/transcript/note
    metadata        JSONB DEFAULT '{}',   -- structured data (tool args, config, etc.)

    -- Search infrastructure
    search_vector   TSVECTOR,            -- BM25 full-text search (trigger-maintained)
    embedding       VECTOR(1024),        -- semantic search (embedding pipeline)

    -- PEFM emotional memory
    pefm_urgency        FLOAT DEFAULT 0.0,
    pefm_insight         FLOAT DEFAULT 0.0,
    pefm_frustration     FLOAT DEFAULT 0.0,
    pefm_satisfaction    FLOAT DEFAULT 0.0,
    pefm_complexity      FLOAT DEFAULT 0.0,
    pefm_novelty         FLOAT DEFAULT 1.0,
    pefm_last_updated    TIMESTAMPTZ DEFAULT NOW(),
    pattern_boost        FLOAT DEFAULT 1.0,

    -- Soft delete
    deleted_at      TIMESTAMPTZ
);
```

## Source Types

| Source | What Goes In | Direction | Example |
|--------|-------------|-----------|---------|
| `chat_log` | Voice/chat transcripts | in/out | User says "fix the fonts" |
| `devlog` | Development log entries | internal | "Refactored voice.py to use GA format" |
| `task_completion` | Task completion events | internal | Task #4370 completed |
| `task_note` | Notes attached to tasks | internal | "Found the root cause in CSS specificity" |
| `task_learning` | Learnings from tasks | internal | "Always check parent selector first" |
| `brain_action` | Brain recommendations | internal | "Recommend dissolving LLC" |
| `sleep_generated` | Sleep cycle insights | internal | "Pattern: user avoids admin on Mondays" |
| `telemetry` | UI/voice events | in/internal | Page view, voice.session_start |
| `sync` | External sync events | in | Claude Code session imported |
| `finance` | Financial events | in | Transaction categorized |

## Tags System

Tags provide flexible categorization. Common tags:

| Tag | Meaning |
|-----|---------|
| `voice` | Came from voice/orb interaction |
| `sleep-generated` | Produced by sleep consolidation |
| `improvement` | Identified as an improvement opportunity |
| `idea` | A new idea or concept |
| `yak-shave` | Tangent captured for later |
| `governance` | Related to governance rules |
| `pattern` | A confirmed behavioral pattern |
| `error` | An error or failure |

## Search: RRF Hybrid Fusion

`reeves.log_search` combines three signals:

1. **BM25 full-text** — `search_vector @@ to_tsquery(query)` — keyword match
2. **Semantic similarity** — `embedding <=> query_embedding` — meaning match
3. **PEFM weighting** — Current emotional relevance after decay

Results are fused using Reciprocal Rank Fusion (RRF):

```
score = Σ (1 / (k + rank_i)) for each signal
```

This means a result that ranks #1 in BM25 and #5 in semantic still scores well, but a result that ranks #1 in both scores much higher.

## Write Patterns

### Telemetry (Voice/UI Events)
```python
from reeves.telemetry import emit, emit_chat

# Generic event
emit('voice.session_start', 'Session started', source='telemetry',
     project_id=project_id, channel='voice')

# Chat transcript
emit_chat(text, role='user', project_id=project_id, channel='voice')
```

Telemetry uses a batched async queue — events are buffered and flushed to the database in batches. No blocking the user.

### Devlog
```python
from reeves.devlog import add_devlog

add_devlog(
    content="Refactored voice.py for GA format",
    tags=['voice', 'improvement'],
    category='engineering'
)
```

### Task Events
```python
from reeves.library import log_task_event

log_task_event(task_id, 'completed', content='Finished the refactor')
```

## Volume

As of Feb 2026:
- 31,625 total rows
- ~84 chat log entries
- ~59 voice-specific entries
- Growing by ~100-500 rows/day depending on activity

## Why One Table

Before the log, Reeves had 14 separate append-only tables: `task_completions`, `task_notes`, `task_learnings`, `devlogs`, `brain_actions`, etc. Each had its own schema, search, and query patterns.

Problems:
- Couldn't search across domains ("find everything related to LLC")
- PEFM had to be computed separately per table
- Sleep cycles had to query 14 tables
- New data types required new tables + new search code

The log collapsed all of this into one primitive. Now:
- `log_search` searches everything at once
- PEFM is computed on every row uniformly
- Sleep cycles query one table
- New data types just need a new `source` value
