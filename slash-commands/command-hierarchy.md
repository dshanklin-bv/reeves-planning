# Command Hierarchy — How They Chain Together

## The Stack

```
/mindreader   -> "Here's what's really going on in your life"
/maxralphie   -> "I've turned that into a product roadmap and built it"
/ralphie      -> "Here are 5 engineers shipping it right now"
```

Each command builds on the previous. Together they form a complete pipeline from understanding to execution.

## Flow

```
         UNDERSTAND          ORCHESTRATE          EXECUTE
        ┌──────────┐       ┌──────────┐       ┌──────────┐
        │          │       │          │       │          │
User -> │mindreader│ ----> │maxralphie│ ----> │ ralphie  │ -> Shipped
        │          │       │          │       │          │
        └──────────┘       └──────────┘       └──────────┘
         Reads mind         Builds vision       Ships features
         Synthesizes        Decomposes           Parallel agents
         Prioritizes        Reviews              Iterates
         Acts (optional)    Tests                Merges
```

## Standalone vs Chained

Each command works standalone:

- `/mindreader` — "Give me a read on my life right now" (no action required)
- `/maxralphie "Add real-time collab"` — "Build this specific vision" (no mindreader needed)
- `/ralphie "Reeves 3.0"` — "Run these existing tasks in parallel" (no vision needed)

But they're most powerful chained:

```
/mindreader --depth god --act
# Output: Vision doc + 8 tasks + 3 message drafts + calendar block

# The vision doc feeds maxralphie:
/maxralphie
# (Auto-discovers from mindreader's output + recent data)
# Output: 6 workstreams with PRDs, reviewed by PAL

# The workstreams feed ralphie:
/ralphie "Reeves 3.0" --max 8
# Output: 8 parallel agents building features, merging, testing
```

## When to Use What

| Situation | Command | Why |
|-----------|---------|-----|
| "What should I work on?" | `/mindreader` | Synthesizes all data into priorities |
| "I know what to build, build it" | `/maxralphie "vision"` | Full product pipeline |
| "I have tasks ready, run them" | `/ralphie "project"` | Parallel execution |
| "Read my mind and ship it" | `/mindreader --act` then `/maxralphie` | Full chain |
| "Quick life check" | `/mindreader --depth shallow` | 2-minute read |
| "Show me the plan but don't build" | `/maxralphie --dry-run` | Preview pipeline |
| "What would the swarm do?" | `/ralphie --dry-run` | Preview execution plan |

## Data Flow Between Commands

### mindreader -> maxralphie

Mindreader produces:
- Prioritized insights (what needs attention)
- Vision document (if `--act` is set)
- Created tasks in Reeves
- Identified patterns and themes

Maxralphie consumes:
- Recent tasks (from mindreader's created tasks)
- Voice transcripts (same data, different lens)
- Vision doc (if present)
- Devlogs (including mindreader's analysis)

### maxralphie -> ralphie

Maxralphie produces:
- Workstreams decomposed from vision
- PRDs for each workstream
- Review feedback (PAL + second opinion)
- Tasks created in Reeves project

Ralphie consumes:
- Tasks from the Reeves project
- PRD files (in `.claude/ralphie-prds/`)
- Review notes (in `.claude/ralphie-reviews/`)
- File dependency map

## The Philosophy

**mindreader** answers: "What does this person need?"

**maxralphie** answers: "How do we build what they need?"

**ralphie** answers: "Ship it. Now."

The person (CEO) provides:
1. Their life (data) — to mindreader
2. One sentence (vision) — to maxralphie (or nothing, it discovers)
3. A project name — to ralphie (or nothing, it finds the latest)

Everything else is handled by the agents.
