# Data Sources — Every Surface Reeves Observes

Reeves builds a model of a person by passively observing the surfaces their life touches. Each data source provides a different signal type.

## Active Data Sources (Connected Today)

### Voice (Orb)
- **Source:** OpenAI Realtime API via WebSocket
- **Signal Type:** Explicit intent, emotional tone, complaints, ideas
- **Storage:** `reeves.log` (source='chat_log', tags='{voice}')
- **Transcription:** GPT-4o-transcribe (user + assistant)
- **Volume:** ~50-100 voice sessions/week
- **Key Insight:** What people say casually to a voice assistant reveals more than what they type formally

### Tasks
- **Source:** Reeves task system (PARA methodology)
- **Signal Type:** Priorities, completion patterns, avoidance patterns
- **Storage:** `reeves.task` table + completion events in `reeves.log`
- **Key Signals:**
  - Created but never started = procrastination
  - Started but abandoned = friction
  - Completed quickly = energy/flow
  - Recurring snoozes = avoidance

### Finance
- **Source:** Mercury API, OFX/QFX file ingest
- **Signal Type:** Spending patterns, account health, obligations
- **Storage:** `finance.*` schema (double-entry journal)
- **Key Signals:**
  - Unusual transactions = something new happening
  - Forgotten subscriptions = financial blind spots
  - Balance trends = stress/health indicator
  - Tax deadlines = time-sensitive obligations

### People (Webb)
- **Source:** Reeves person database + knowledge graph
- **Signal Type:** Relationship health, contact frequency, network topology
- **Storage:** `reeves.persons` + `reeves.relationship_edges`
- **Key Signals:**
  - Decreasing contact frequency = relationship drift
  - Unanswered messages = dropped threads
  - Network clusters = who's connected to what

### Messages (Tosh)
- **Source:** Apple Messages via Tosh MCP
- **Signal Type:** Relationship priorities, emotional tone, commitments
- **Access:** Real-time search + contact resolution
- **Key Signals:**
  - Most frequent contacts = relationship priorities
  - Unanswered messages = obligations
  - Emotional tone analysis = mood patterns
  - Promises made = broken promise detection

### Calendar (Outlook)
- **Source:** Outlook MCP (browser automation)
- **Signal Type:** Time commitments, scheduling patterns, overload signals
- **Key Signals:**
  - Meeting density = capacity indicator
  - Rescheduled meetings = avoidance signal
  - Free blocks = opportunity windows
  - Upcoming deadlines = urgency triggers

### Devlogs
- **Source:** Reeves devlog system
- **Signal Type:** Engineering decisions, debugging patterns, ideas
- **Storage:** `reeves.log` (source='devlog')
- **Key Signals:**
  - Tagged `improvement` or `idea` = latent ambition
  - Debugging sessions = pain points
  - Architecture decisions = values signal

### Browser/UI Telemetry
- **Source:** Reeves web UI telemetry.js
- **Signal Type:** Behavioral patterns, feature usage, time-of-day activity
- **Storage:** `reeves.log` (source='telemetry')
- **Key Signals:**
  - Pages visited most = obsessive checking
  - Features never used = what's not working
  - Time patterns = when active/productive

## AI-Generated Data Sources

These are produced by Reeves' own intelligence systems:

### PEFM (Emotional Memory)
- **Source:** Computed on every log write
- **Signal Type:** Weighted emotional dimensions
- **6 Dimensions:** Urgency, Insight, Frustration, Satisfaction, Complexity, Novelty
- **Decay:** Each dimension has a different half-life (6hr to 90d)
- **Key Insight:** Memories are never deleted, just archived to near-zero relevance

### Sleep Briefings
- **Source:** Memory consolidation daemon (3-stage: Light/Deep/REM)
- **Signal Type:** Cross-domain patterns, anomalies, themes
- **Storage:** `reeves.log` (tags='{sleep-generated}')
- **Frequency:** Configurable cycles, runs in background
- **Key Insight:** Finds patterns humans can't see by cross-referencing all data sources

### Brain Recommendations
- **Source:** Brain module (adaptive steering with approval workflows)
- **Signal Type:** Suggested actions, confidence levels, pending approvals
- **Key Insight:** AI intuition about what you should work on

### Knowledge Graph
- **Source:** Graph module (RDF triples)
- **Signal Type:** Relationship topology, temporal gaps, connection strength
- **Storage:** `reeves.relationship_edges`
- **Key Insight:** Network effects — who connects to what, and what patterns emerge

### Embeddings
- **Source:** pgvector (1024-dim, all entity types)
- **Signal Type:** Semantic similarity across all content
- **Storage:** `embeddings.knowledge_vectors`
- **Key Insight:** "Things that are similar" across all domains

### Intents & Facts
- **Source:** User-stated or AI-inferred
- **Signal Type:** Goals, values, preferences, constraints, biographical data
- **Storage:** `reeves.intents` + `reeves.facts`
- **Key Insight:** What the person wants to be true about their life

## Data Source Matrix

| Source | Explicit | Behavioral | Emotional | Shadow |
|--------|----------|-----------|-----------|--------|
| Voice | High | Low | Medium | Low |
| Tasks | Medium | High | Low | High |
| Finance | Low | High | Low | Medium |
| Messages | Medium | Medium | High | Medium |
| Calendar | Medium | High | Low | Medium |
| Devlogs | High | Medium | Low | Low |
| Telemetry | Low | High | Low | Medium |
| PEFM | — | — | High | — |
| Sleep | — | — | — | High |
| Brain | — | — | — | Medium |
| Graph | Low | Medium | Low | High |

**Shadow signal** = what you learn from absence (tasks not done, people not contacted, money not reviewed).

## Future Data Sources (Not Yet Connected)

- **Health wearables** — Heart rate, sleep quality, activity levels
- **Photos** — Visual context (who you're with, where you go)
- **Music** — Mood indicators (what you listen to when)
- **Location** — Movement patterns, places frequented
- **Browser history** — Research topics, interests, rabbit holes
- **Code commits** — Engineering productivity patterns

Each new data source adds another dimension to the person model without requiring any configuration from the user.
