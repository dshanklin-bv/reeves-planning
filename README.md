# Reeves Planning

The gold standard for building a personal AI operating system that reads minds and collapses the distance between intent and outcome.

## Read This First

Reeves is a macOS-native personal AI system. It observes your life through data sources only a Mac can access (Messages, Photos, Screen Time, Notes), models who you are, infers what you need, and collapses the distance between intent and outcome.

**The cardinal rule: everything goes through the daemon.** All services, jobs, and processes are managed by `reeves-daemon` via `launchd`. Do not install services globally, run processes manually, or create standalone launchd plists. If you're adding a new process to any service, define it in the daemon's configuration.

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

## Repo Structure

```
reeves-planning/
│
├── VISION.md                          # Core philosophy and north star
│
├── specs/                             # SPECS — executable specifications for agent work
│   ├── autonomous-bookkeeping.md      #   AI does the books: pipeline, enrichment, LLM, reporting
│   └── financial-entity-model.md      #   Accounts, institutions, merchants, payee normalization
│
├── methodology/                       # HOW — development planning philosophy
│   ├── README.md                      #   When to use conga vs skateboard, spec lifecycle, three rules
│   ├── elephant-conga-carpaccio.md    #   Dependency chains sliced thin
│   ├── elephant-carpaccio.md          #   Alistair Cockburn's original principle
│   └── meta-cognition.md             #   Plan→Act→Reflect→Refine loop, PAL as meta-agent
│
├── philosophy/                        # WHY — the intellectual foundation
│   ├── essays/                        #   AI hasn't lived yet, Krebs cycle, 750MB, write DNA not code
│   ├── frameworks/                    #   Two primitives, evolutionary loop, selection pressure
│   └── implications/                  #   Democratization of agency, scaling isn't enough, timeline
│
├── architecture/                      # WHAT — the product design
│   ├── mind-reading-loop.md           #   The 6-step continuous intelligence loop
│   ├── reeves-3.0-pillars.md         #   5 engineering pillars
│   ├── data-sources.md               #   Every surface Reeves observes
│   ├── agent-roster.md              #   Claude, PAL, Ralph Loop — the product org
│   ├── router.md                     #   Capability discovery: agents ask, router answers
│   ├── deep-sleep.md                 #   LEARN machinery: consolidation pipeline, tiered LLM costs
│   └── governance.md                 #   Three pillars: context, intent, guardrails
│
├── infrastructure/                    # HOW — how it runs on the Mac
│   ├── platform.md                    #   Mac-only thesis, hardware, data flow, why not Linux
│   ├── daemon.md                      #   reeves-daemon: launchd, service lifecycle, state machine
│   ├── gateway.md                     #   Caddy reverse proxy, UDS, path-based routing
│   ├── services.md                    #   Anatomy of a service (web + CLI + MCP + jobs)
│   ├── common.md                      #   reeves-common: make_app, shared CSS, health endpoints
│   ├── mechanic.md                    #   Fleet health, integration testing, compliance scanners
│   └── processes.md                   #   Multi-process architecture: jobs, daemons, socket activation
│
├── schemas/                           # DATA — the unified primitives
│   ├── log-table.md                   #   Unified append-only log (31K+ rows, PEFM, embeddings)
│   ├── pefm.md                        #   Personal Emotional Field Model (6 dimensions, decay)
│   ├── knowledge-graph.md             #   RDF triples with confidence
│   └── person-model.md              #   What "knowing a person" means
│
├── slash-commands/                    # AGENTS — autonomous command specs
│   ├── mindreader.md                  #   /mindreader — know me better than I know myself
│   ├── maxralphie.md                  #   /maxralphie — vision accelerator (15-25 agents)
│   ├── ralphie.md                     #   /ralphie — parallel task swarm
│   └── command-hierarchy.md           #   How they chain together
│
├── research/                          # RESEARCH — capabilities and learnings
│   ├── opus-4.6-capabilities.md       #   Claude Opus 4.6 benchmarks + features
│   ├── claude-code-features.md        #   CLI capabilities for autonomous swarms
│   ├── pal-mcp-tools.md             #   Multi-model orchestration
│   ├── openai-realtime-ga.md         #   Voice API debugging learnings
│   └── voice-transcripts.md          #   59 recovered voice messages
│
├── tasks/                             # TASKS — project tracking
│   ├── reeves-3.0-project.md         #   11 tasks across 5 pillars
│   └── ux-issues-from-voice.md       #   Issues surfaced from voice transcripts
│
└── context/                           # CONTEXT — who's building this
    └── who-is-dallas.md              #   Builder profile + societal ambition
```

## Development Methodology

See `methodology/` for the full planning philosophy. The short version:

1. **Two decomposition strategies.** Pipelines use **Elephant Conga Carpaccio** (dependency chains, sliced thin). Products use **Skateboard → Bike → Car** (standalone versions, increasing fidelity). See `methodology/README.md` for when to use which.

2. **Spec-Driven Development.** Write executable specs before code. Specs define behavioral scenarios, acceptance criteria, commands, and boundaries. Agents pick them up cold and execute. Specs live in `specs/`.

3. **Everything Through the Daemon.** All services, jobs, and processes managed by `reeves-daemon` via `launchd`. No standalone plists, no global packages, no manual processes.

4. **Run real data after every slice.** The slice isn't done when the code compiles. It's done when you run it against reality and see what it does.

### How to Navigate

- **New to reeves?** Start with `VISION.md`, then `philosophy/essays/ai-hasnt-lived-yet.md`.
- **Building a service?** Read `infrastructure/services.md` and `infrastructure/common.md`.
- **Working on the daemon?** Read `infrastructure/daemon.md`, `infrastructure/processes.md`, and `infrastructure/gateway.md`.
- **Understanding the data model?** Read `schemas/log-table.md` — it's the unified primitive everything else builds on.
- **AI session starting work?** Read this README and `infrastructure/services.md` before touching any code.

## Origin

This repo consolidates the vision, philosophy, architecture, and infrastructure documentation for the Reeves ecosystem. The vision and philosophy crystallized from a design conversation on February 14, 2026. The infrastructure documentation captures the running system as it evolves.
