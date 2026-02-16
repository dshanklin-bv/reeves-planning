# Deep Sleep: The LEARN Machinery

The loop says "outcomes feed back, model improves, cycle repeats." Deep Sleep is how.

## What It Does

Raw observations accumulate in the unified log table. Deep Sleep consolidates them into durable knowledge: graph mutations, PEFM recalibrations, pattern rules, and cross-domain insights.

## The Pipeline

```
Raw log entries (31K+ rows)
  → Light Sleep: summarize, deduplicate, tag
  → Deep Sleep: extract patterns, cross-reference domains
  → REM Sleep: generate cross-domain insights, update knowledge graph
  → Human gate: approve graph mutations and new rules
  → Model updated
```

Each stage is a daemon job managed by the daemon (see `processes.md`). Each stage has a staging table with lifecycle: `pending → triaged → approved → ingested`.

## What Each Stage Produces

### Light Sleep (Tier 1 — cheap LLM)
- Session summaries (one-line captures of what happened)
- Devlog categorization (feature/bugfix/architecture/decision)
- Deduplication (same insight captured multiple times → merge)
- Tag enrichment (auto-tag entries with relevant domains)

### Deep Sleep (Tier 2 — expensive LLM)
- Pattern extraction: "Daniel texts his mom every Sunday" from 3 months of message data
- Anomaly detection: "Finance spending up 40% this month vs trailing average"
- Relationship scoring recalculation from interaction frequency changes
- PEFM dimension recalibration based on outcome data

### REM Sleep (Tier 2 — cross-domain)
- Cross-domain connections: "The Hilton charges correlate with Nashville meetings in calendar"
- Broken promise detection: things said in messages without corresponding task completions
- Knowledge graph mutations: new edges, updated confidence scores
- Categorization rule generation from approved transaction patterns

## Human Gates

Graph mutations and new rules require approval before they change the model. This prevents hallucinated patterns from corrupting the knowledge base.

```
Deep Sleep proposes: "Daniel → works_with → Sarah (confidence: 0.7)"
  Evidence: 3 email threads, 2 calendar meetings, 1 message mention
  → Human reviews evidence → approves/rejects
  → If approved: edge created, confidence logged
```

## Tiered Cost Model

Borrowed from taskr-worker's proven pattern:

| Tier | What | Cost | Frequency |
|------|------|------|-----------|
| 0 | Data sync, dedup, staging | Free | Continuous |
| 1 | Summarization, tagging, categorization | ~$0.0001/1K tokens | Hourly |
| 2 | Pattern extraction, cross-domain reasoning | $0.50-2.00/run | Daily |

## Relationship to taskr

taskr-worker built this pattern first for the AIC ecosystem. Reeves inherits the three-tier architecture and the staging pipeline. The difference: taskr consolidates engineering knowledge (devlogs, sessions). Reeves consolidates life knowledge (transactions, messages, health, relationships).
