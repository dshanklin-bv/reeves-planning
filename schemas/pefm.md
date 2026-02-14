# PEFM — Personal Emotional Field Model

## What It Is

PEFM is an emotional memory system that weights every piece of information by how emotionally relevant it is *right now*. Unlike traditional timestamps or priority scores, PEFM mimics how human memory actually works: urgent things feel important for hours, insights last for months, novelty fades in days.

Every row in `reeves.log` carries 6 PEFM dimensions. When you search, results are weighted by current emotional relevance, not just recency or keyword match.

## The Six Dimensions

| Dimension | Half-Life | What It Captures | Default Boost |
|-----------|-----------|------------------|---------------|
| **Urgency** | 6 hours | Time-sensitive tasks, deadline pressure | 1.0 |
| **Insight** | 90 days | Valuable wisdom, architectural understanding | 1.0 |
| **Frustration** | 3 days | Hard-won debugging knowledge, pain points | 1.0 |
| **Satisfaction** | 14 days | Success memories, wins, completions | 1.0 |
| **Complexity** | 7 days | Technical depth, intricate problem-solving | 0.5 |
| **Novelty** | 2 days | New discoveries, first encounters | 1.0 |

## Decay Model

Each dimension decays exponentially:

```
current_value = initial_value * 2^(-hours_elapsed / half_life)
```

Example for urgency (half-life = 6 hours):
- t=0: 1.0 (just happened)
- t=6h: 0.5 (half as urgent)
- t=12h: 0.25 (quarter as urgent)
- t=24h: 0.0625 (barely relevant)
- t=48h: 0.00390625 (archived to near-zero)

Example for insight (half-life = 90 days):
- t=0: 1.0
- t=90d: 0.5 (still half-relevant after 3 months)
- t=180d: 0.25
- t=1yr: 0.063 (still a faint signal)

Memories are never deleted. They just fade to near-zero relevance.

## Database Schema

Stored directly on the `reeves.log` table:

```sql
-- PEFM columns on reeves.log
pefm_urgency       FLOAT DEFAULT 0.0 NOT NULL,
pefm_insight       FLOAT DEFAULT 0.0 NOT NULL,
pefm_frustration   FLOAT DEFAULT 0.0 NOT NULL,
pefm_satisfaction   FLOAT DEFAULT 0.0 NOT NULL,
pefm_complexity    FLOAT DEFAULT 0.0 NOT NULL,
pefm_novelty       FLOAT DEFAULT 1.0 NOT NULL,  -- New things start novel
pefm_last_updated  TIMESTAMPTZ DEFAULT NOW(),
pattern_boost      FLOAT DEFAULT 1.0 NOT NULL,   -- Multiplier for confirmed patterns
```

## How PEFM Is Set

### On Write (Automatic)
When a log entry is created, PEFM values are set based on the event type:

| Event Type | Urgency | Insight | Frustration | Satisfaction |
|------------|---------|---------|-------------|-------------|
| Task completion | Low | Low | Low | High |
| Bug fix | Medium | Medium | High | Medium |
| Voice complaint | High | Low | High | Low |
| Architecture decision | Low | High | Low | Medium |
| Sleep insight | Low | High | Low | Low |
| Deadline approaching | High | Low | Medium | Low |

### On Boost (Manual/AI)
PEFM dimensions can be boosted by:
- Brain recommendations (urgency boost on stalled tasks)
- Sleep cycles (insight boost on discovered patterns)
- User interactions (frustration boost on repeated complaints)
- Completion events (satisfaction boost, urgency decay)

### Pattern Boost
The `pattern_boost` multiplier amplifies entries that match confirmed patterns:
- Default: 1.0
- Confirmed pattern: 1.5-2.0
- Strongly reinforced: 2.0-3.0

## How PEFM Is Queried

### Search Weighting
The `log_search` function uses PEFM to weight results:

```
relevance = text_score * (1 + urgency_now + insight_now + frustration_now) * pattern_boost
```

This means:
- A 6-hour-old urgent message scores higher than a 2-day-old one
- A 3-month-old architectural insight still shows up in relevant searches
- A frustrating debugging session stays relevant for ~3 days
- Confirmed patterns get boosted across all dimensions

### Mindreader Usage
`/mindreader` queries PEFM to find:
- High urgency items (time-sensitive)
- High frustration items (pain points)
- Rising urgency + zero actions (procrastination signature)
- Decaying satisfaction (wins that are fading)

## Why PEFM Exists

Traditional search weights by recency or keyword match. Neither captures what actually matters to a person right now.

A debugging session from 2 days ago is more relevant than a task created an hour ago if you're still frustrated about the bug. An architectural insight from 2 months ago is more relevant than yesterday's standup notes.

PEFM makes search feel like memory: important things stay accessible, routine things fade, and emotional intensity determines persistence.

## Implementation

Source: `reeves/pefm.py` (466 lines)

Key functions:
- `compute_pefm_dimensions(source, event_type, content)` — Set initial PEFM on write
- `boost_pefm(log_id, dimension, amount)` — Boost a specific dimension
- `get_current_pefm(log_id)` — Calculate decayed values at query time
- `get_emotional_state(entity_type, entity_id)` — Aggregate PEFM across all related logs
