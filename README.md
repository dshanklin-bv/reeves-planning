# Reeves Planning

The gold standard for building a personal AI operating system that reads minds and collapses the distance between intent and outcome.

## What This Is

This repo contains the complete planning, vision, architecture, and research for **Reeves 3.0** — transforming a powerful-but-invisible AI system into the first true personal AI operating system. One that observes your life, models who you are, infers what you need, and builds the outcome before you ask.

## The Philosophy

AI should collapse the distance between intent and outcome for everyone. Not just for coders. Not just for the technically capable. For anyone who can describe what they want and let intelligence handle the rest.

Reeves is the prototype. Built for one person first — the most instrumented test case available. Every pattern validated here becomes architecture that works for everyone.

## Repo Structure

```
reeves-planning/
├── VISION.md                         # Core philosophy and north star
├── architecture/
│   ├── mind-reading-loop.md          # The 6-step continuous intelligence loop
│   ├── reeves-3.0-pillars.md        # 5 engineering pillars
│   ├── data-sources.md              # Every surface Reeves observes
│   └── agent-roster.md             # PAL, Ralph, Claude — the product org
├── slash-commands/
│   ├── mindreader.md                # /mindreader — know me better than I know myself
│   ├── maxralphie.md                # /maxralphie — vision accelerator
│   ├── ralphie.md                   # /ralphie — parallel task swarm
│   └── command-hierarchy.md         # How they chain together
├── schemas/
│   ├── pefm.md                      # Personal Emotional Field Model
│   ├── knowledge-graph.md           # RDF triple graph with confidence
│   ├── log-table.md                 # Unified log primitive
│   └── person-model.md             # What "knowing a person" means
├── research/
│   ├── opus-4.6-capabilities.md     # Claude Opus 4.6 benchmarks + features
│   ├── claude-code-features.md      # CLI capabilities for autonomous swarms
│   ├── pal-mcp-tools.md            # Multi-model orchestration
│   ├── openai-realtime-ga.md       # Voice API debugging learnings
│   └── voice-transcripts.md        # 59 recovered voice messages
├── context/
│   ├── who-is-dallas.md            # Builder profile + societal ambition
│   └── chat-transcript.jsonl       # Full design conversation for study
└── tasks/
    ├── reeves-3.0-project.md       # All tasks from the Reeves 3.0 project
    └── ux-issues-from-voice.md     # Issues surfaced from voice transcripts
```

## The Loop

```
OBSERVE   →  passive collection from life surfaces
MODEL     →  PEFM + graph + embeddings build a person model
INFER     →  gap between words and actions reveals true intent
COLLAPSE  →  system does maximum work, person does minimum
GOVERN    →  testing and constraints make speed safe
LEARN     →  outcomes feed back, model improves, cycle repeats
```

## The Command Chain

```
/mindreader   →  "Here's what's really going on in your life"
/maxralphie   →  "I've turned that into a product roadmap and built it"
/ralphie      →  "Here are 5 engineers shipping it right now"
```

## Origin

This repo was created from a single design conversation on February 14, 2026. The full transcript is preserved in `context/chat-transcript.jsonl` for study. Everything here — the vision, the architecture, the slash commands, the schemas — emerged from that conversation.
