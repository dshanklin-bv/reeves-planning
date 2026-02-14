# The Mind-Reading Loop

The 6-step continuous intelligence loop that drives everything Reeves does.

## The Loop

```
OBSERVE   ->  passive collection from life surfaces
MODEL     ->  PEFM + graph + embeddings build a person model
INFER     ->  gap between words and actions reveals true intent
COLLAPSE  ->  system does maximum work, person does minimum
GOVERN    ->  testing and constraints make speed safe
LEARN     ->  outcomes feed back, model improves, cycle repeats
```

## Step 1: OBSERVE — The Person Just Lives

The input isn't a prompt. It's a life. Voice conversations. Bank transactions. Text messages. Calendar. What they click on. What they avoid. What makes them laugh. What stresses them out.

The system collects this passively. The person doesn't "configure" anything. They just exist near the system.

### Data Collection Points

| Surface | What's Collected | How |
|---------|-----------------|-----|
| Voice (Orb) | Everything said to Reeves | OpenAI Realtime API transcription |
| Messages (Tosh) | iMessage conversations | Apple Messages sync via Tosh MCP |
| Finance | Transactions, balances | Mercury API, OFX/QFX ingest |
| Calendar | Events, scheduling patterns | Outlook MCP |
| Tasks | What's done, what's avoided | Reeves task system |
| Browser | Pages visited, time patterns | Telemetry events |
| Devlogs | Engineering decisions | Claude Code sessions |

## Step 2: MODEL — The Model Builds Itself

**PEFM (Personal Emotional Field Model)** generates emotional signal from every interaction:

| Dimension | Half-Life | What It Captures |
|-----------|-----------|------------------|
| Urgency | 6 hours | Time-sensitive tasks, decays fast |
| Insight | 90 days | Valuable wisdom, persists long |
| Frustration | 3 days | Hard-won debugging knowledge |
| Satisfaction | 14 days | Success memories |
| Complexity | 7 days | Technical depth |
| Novelty | 2 days | New discoveries lose freshness |

**Knowledge Graph** grows from observation, not configuration:
- "This person texts their mom every Sunday" extracted from message patterns during a sleep cycle
- "This person avoids admin tasks in build phases" mined from task completion data cross-referenced with voice session frequency
- RDF triples (subject-predicate-object) with confidence scores

**Embeddings** (1024-dim vectors via pgvector) enable semantic search across all entity types. Content is hashed to prevent duplicate computation.

After enough cycles, the system has a model of the person that the person never explicitly created.

## Step 3: INFER — Intent Is Inferred, Not Stated

The person doesn't say "I need to dissolve my LLC." They mention it to Reeves while talking about something else. Four times over three weeks. The system notices:

- Mentioned 4 times
- Zero actions taken
- PEFM urgency rising then decaying (procrastination signature)
- Deadline approaching

The system now knows something the person hasn't fully admitted to themselves: this is important, they're avoiding it, and it's going to become urgent whether they act or not.

### Inference Sources

1. **Procrastination signatures** — Tasks mentioned but never started (high mention count, zero completions)
2. **Relationship drift** — People in the graph with decreasing contact frequency
3. **Financial blind spots** — Recurring charges never reviewed, accounts with low activity
4. **Energy misallocation** — Time on low-PEFM tasks while high-PEFM tasks are avoided
5. **Broken promises** — Things said to people (from messages/voice) without corresponding tasks

## Step 4: COLLAPSE — The Distance Disappears

The system doesn't say "you should dissolve your LLC." It says:

> "I've broken the dissolution into 5 steps. The first one takes 10 minutes. The form is pre-filled. I blocked 30 minutes tomorrow morning. The only thing you need to do is sign and mail."

The distance between "I need to do this thing I've been avoiding" and "it's basically done" went from infinite (procrastination loop) to 10 minutes of human effort.

### How Collapse Works in Reeves

| Signal | System Response |
|--------|----------------|
| Task mentioned 4x, never started | Break into subtasks, schedule first one |
| Relationship going cold | Draft message, present for approval |
| Subscription forgotten | Surface in finance review with cancel option |
| Build energy high, admin avoided | Schedule 30-min admin block before build time |

## Step 5: GOVERN — Safety Is Structural

Every action has a governance gate. Not a "are you sure?" dialog. A test suite:

- **Brain** (`brain.py`) evaluates actions against constraints and recommends approval/denial
- **939+ automated tests** prevent regressions
- **Sleep cycles** extract patterns from failures and encode them as guardrails
- **Soft deletes only** — nothing is permanently destroyed
- **Human review gates** for financial transactions, relationship actions, external communications

Safety is invisible when it's structural. The person never sees the governance. They just notice that things work.

## Step 6: LEARN — The System Gets Smarter

Every cycle feeds back:

- Did the person actually do the task? If yes: strategy gets reinforced
- If no: the system learns that too. Maybe the barrier wasn't time — it was anxiety. Next approach: surface the emotional block
- **PEFM dimensions decay** at different rates, naturally prioritizing recent and impactful memories
- **Sleep consolidation** (Light/Deep/REM cycles) extracts cross-domain patterns
- **Pattern boost** multiplies relevance for confirmed patterns
- **Devlog** captures engineering learnings that persist across all sessions

The model gets better every day. Not because someone tunes it. Because it observes outcomes and adjusts.

## The Generalization

Replace one person with anyone. Replace Reeves' specific data sources with whatever surfaces a person's life touches. The architecture is the same.

The single mom in Fort Worth doesn't need to know what PEFM is. She talks to her phone: "I need groceries, we're gluten-free, I have $100, and my kid won't eat broccoli." Six cycles later, the system knows her shopping patterns, her kids' preferences, her budget rhythm. She doesn't order groceries anymore. They just arrive.

That's not a productivity tool. That's a new relationship between humans and complexity.
