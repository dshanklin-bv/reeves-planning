# Agent Roster — The Product Organization

Reeves 3.0 operates as a full product organization. Each agent has a specific role, strengths, and when it's deployed.

## The Hierarchy

```
User (CEO)
  |
  ├── /mindreader (Chief of Staff)
  |     Knows what you need before you ask.
  |     Reads all data sources, synthesizes the picture.
  |
  ├── /maxralphie (VP of Engineering)
  |     Turns the vision into a product roadmap.
  |     Orchestrates the entire build pipeline.
  |
  └── /ralphie (Engineering Manager)
        Runs 5-10 engineers in parallel.
        Ships features, resolves conflicts, closes loops.
```

## Individual Agents

### Claude (Opus 4.6) — Core Intelligence

**Role:** Primary reasoning engine, code generation, synthesis

**Capabilities:**
- 1M token context window
- 128K output tokens (long generation)
- 65.4% Terminal-Bench (best autonomous coding)
- Extended thinking for hard problems
- Self-correcting loops
- 10 parallel subagents via Task tool

**Deployed By:** Everything. Claude is the substrate.

**Strengths:** Nuanced reasoning, code generation, long-context synthesis, tool orchestration

**Limitations:** Single model perspective (why PAL exists), may not catch its own blind spots

### PAL (GPT-5.2 + Others) — Multi-Model Orchestrator

**Role:** Second perspective, specialized reviews, strategic guidance

**Available via:** `mcp__pal__*` tools (15 tools)

**Sub-Agents:**

| Tool | Model | Purpose |
|------|-------|---------|
| `chat` | Configurable | General conversation with any model |
| `planner` | GPT-5.2 | Architecture decomposition, roadmap planning |
| `codereview` | GPT-5.2 | Technical code review |
| `secaudit` | GPT-5.2 | Security vulnerability scanning |
| `precommit` | GPT-5.2 | Code quality gate before commit |
| `debug` | GPT-5.2 | Debugging assistance |
| `testgen` | GPT-5.2 | Test generation |
| `docgen` | GPT-5.2 | Documentation generation |
| `analyze` | GPT-5.2 | Code analysis |
| `refactor` | GPT-5.2 | Refactoring suggestions |
| `challenge` | GPT-5.2 | Devil's advocate on decisions |
| `consensus` | Multi-model | Multiple models vote on approach |
| `thinkdeep` | GPT-5.2 | Extended reasoning on hard problems |
| `apilookup` | GPT-5.2 | API documentation lookup |

**Why PAL Matters:** Different training data = different blind spots. PAL catches what Claude misses. Architecture decisions benefit from disagreement.

### Ralph Loop Agents — Iterative Engineers

**Role:** Build features through self-correcting iteration

**Mechanism:** Same prompt fed repeatedly. Agent sees its own previous work in files. Each iteration improves on the last.

**Deployed By:** `/ralphie` (5-10 parallel) or `/maxralphie` (15-25 across pipeline)

**Key Properties:**
- Branch isolation prevents conflicts between parallel agents
- Max iterations per agent (10 for build, 3 for fix)
- Completion signaled by `<promise>` tag
- File scope constraints prevent unintended changes

### Reeves Brain — Adaptive Steering

**Role:** Recommend actions, manage approval workflows, enforce governance

**Located at:** `reeves/brain.py` (898 lines)

**Key Functions:**
- `brain.think()` — Evaluate situation and recommend actions
- Approval workflows — Human review gates for risky actions
- Constraint checking — Validate against intents and governance rules
- Confidence calibration — Track recommendation accuracy

### Sleep Daemon — Memory Consolidation

**Role:** Extract patterns from behavior, consolidate learnings, detect anomalies

**Located at:** `reeves/sleep.py`

**Three Stages:**
1. **Light sleep** — Quick pattern scan, recent activity summary
2. **Deep sleep** — Cross-domain analysis, relationship mapping
3. **REM sleep** — Creative connections, anomaly detection, insight generation

**Output:** Sleep briefings stored in `reeves.log` (tags='{sleep-generated}')

### Triage — What to Work On

**Role:** Prioritize tasks based on urgency, impact, and context

**Located at:** `reeves/triage.py`

**Inputs:** Task backlog, PEFM state, brain recommendations, recent activity

**Output:** Prioritized task list with justification

## How They Chain Together

### Mindreader Pipeline (Synthesis)
```
mindreader
  ├── 8 parallel data gatherers (voice, tasks, finance, people, PEFM, calendar, sleep, graph)
  ├── PAL planner (prioritization)
  ├── PAL consensus (multi-model validation)
  └── Output: "Here's what's going on in your life"
```

### Maxralphie Pipeline (Orchestration)
```
maxralphie
  ├── Phase 0: Mine data (reuse mindreader's data layer)
  ├── Phase 1: PAL planner (decompose into workstreams)
  ├── Phase 2: 5-8 Claude agents (write PRDs in parallel)
  ├── Phase 3: PAL codereview + PAL challenge (review in parallel)
  ├── Phase 4: 5-8 Ralph Loop agents (build in parallel)
  ├── Phase 5: Test + PAL secaudit + PAL precommit (QA in parallel)
  └── Phase 6: Merge, report, devlog
```

### Ralphie Pipeline (Execution)
```
ralphie
  ├── Gather tasks from Reeves project
  ├── Analyze file conflicts, group into parallel slots
  ├── Spawn 5-10 background agents
  ├── Monitor every 60 seconds
  └── Mark complete, devlog results
```

## Total Agent Capacity

| Scenario | Active Agents | Duration |
|----------|--------------|----------|
| `/mindreader --depth shallow` | 3-5 | 2 minutes |
| `/mindreader --depth god --act` | 10-12 | 10 minutes |
| `/ralphie` (5 tasks) | 5-7 | 10-20 minutes |
| `/maxralphie` (full pipeline) | 15-25 | 30-60 minutes |

All agents log their work to devlog. Nothing is lost. Every decision, every build, every review becomes durable context for future sessions.
