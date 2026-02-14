# Maxralphie — Vision Accelerator (Full Product Org in Parallel)

Maxralphie doesn't need you to describe the vision. **Reeves already heard it.** You said it through the orb. You said it in devlogs. You said it in task descriptions. Maxralphie's job is to listen to what you've already said, synthesize the signal, and execute the entire product development pipeline — headless, parallel, walk-away.

## The Hierarchy

```
/ralphie    = Foreman    — runs tasks in parallel within your session
/maxralphie = CEO mode   — discovers what to build from your data, then builds it
```

## Arguments

`/maxralphie [PROJECT_OR_VISION] [--max N] [--skip PHASE] [--dry-run] [--since DURATION]`

- **PROJECT_OR_VISION** — Optional. A Reeves project name, vision statement, or NOTHING (auto-discover from recent data)
- **--max N** — Max concurrent agents (default: 8, max: 10)
- **--skip PHASE** — Skip a phase (listen, prd, review, build, test, qa)
- **--since DURATION** — How far back to mine for signal (default: 7d)
- **--dry-run** — Show the full pipeline plan without launching

If no argument is given, maxralphie mines recent voice transcripts, devlogs, and tasks to discover what needs building.

---

## The Pipeline

### Phase 0: Listen (Mine Your Own Data for the Vision)

Before decomposing anything, maxralphie discovers WHAT to build by mining Reeves' own data.

**Data sources (queried in parallel):**

1. **Voice transcripts** (reeves.log WHERE source='chat_log' AND tags @> '{voice}'):
   - User complaints ("the fonts don't stick")
   - Feature requests ("I need a working on it indicator")
   - UX observations ("it's too choppy")
   - Ideas mentioned in passing ("what if the orb changed color when...")

2. **Devlogs** (reeves_devlog_search):
   - Recent debugging sessions (what broke, what was learned)
   - Improvement priorities identified in analysis
   - Architecture decisions made but not yet implemented
   - Patterns tagged with `improvement` or `idea`

3. **Task backlog** (reeves_task_list):
   - High-priority tasks not yet started
   - Tasks with detailed instructions (ready to build)
   - Tasks tagged with project themes (pillar-1, pillar-2, etc.)

4. **Brain suggestions** (brain.think):
   - What does the brain recommend working on?
   - Are there pending approvals blocking progress?

5. **Sleep briefings** (reeves.log WHERE tags @> '{sleep-generated}'):
   - Latest patterns extracted during consolidation
   - Cross-project themes identified
   - Anomalies worth addressing

**Synthesis agent: PAL (planner)**

Feed all mined data into PAL's planner:
```
"Here is raw signal from the last {since} days of a personal AI operating system.
 Voice transcripts (user feedback), devlogs (engineering learnings),
 task backlog (known work), brain suggestions (AI recommendations),
 sleep briefings (consolidated patterns).

 Synthesize this into a prioritized product roadmap:
 - What are the top 5-8 things that should be built?
 - Why? (trace each to specific user feedback or data)
 - In what order? (dependencies, impact, effort)
 - What should NOT be built? (scope control)"
```

PAL returns a structured roadmap. Each item becomes a workstream.

**If a project name IS provided:** Skip mining, pull existing tasks. But still check voice transcripts for any untracked feedback about that project.

**If a vision statement IS provided:** Use it as seed, but augment with mined data for context and constraints.

**Why Phase 0 matters:** This is the difference between "tool that does what you say" and "system that knows what you need." The voice transcripts are product research. The devlogs are engineering intelligence. The brain suggestions are AI intuition. Maxralphie uses all of it.

### Phase 1: Vision Decomposition (PAL as Architect)

If given a project name, pull existing tasks. If given a vision statement, decompose it.

**Agent: PAL (GPT-5.2 via mcp-pal)**

```
Use PAL's `planner` tool:
  "I'm building {vision}. Here's the codebase structure: {tree}.
   Decompose this into 5-8 independent workstreams.
   For each: name, goal, files to modify, dependencies, acceptance criteria.
   Flag any workstreams that conflict on files."
```

PAL returns structured workstreams. Each becomes a task in Reeves if it doesn't exist yet.

**Why PAL here:** PAL brings a different model's perspective. It catches blind spots Claude has. Different training data, different intuitions. Architecture decisions benefit from disagreement.

### Phase 2: PRD Generation (Parallel — 1 per workstream)

For each workstream, spawn a background agent that writes a PRD:

**Agent: Claude via `claude -p`**

Each agent writes `.claude/ralphie-prds/{task_id}.md`:

```markdown
# PRD: {workstream_name}

## Problem
What user pain does this solve? Why now?

## Solution
Specific implementation approach. Files to modify. API changes.

## Acceptance Criteria
- [ ] Criteria 1 (testable)
- [ ] Criteria 2 (testable)
- [ ] Criteria 3 (testable)

## Technical Design
- Data flow
- Component changes
- Edge cases

## Out of Scope
What we're NOT doing (prevents scope creep)

## Dependencies
What must be true before this can start?
```

**All PRDs generated in parallel** — one `claude -p` per workstream.

### Phase 3: Architecture Review (PAL + Second Opinion)

Once PRDs exist, run two reviewers in parallel:

**Agent A: PAL Code Review**
```
Use PAL's `codereview` tool:
  Review these {N} PRDs for: conflicts, missing edge cases,
  security concerns, performance gotchas. Flag anything that
  would break existing tests or violate project conventions.
```

**Agent B: Second Opinion (via Janus)**
```
Use janus_second_opinion:
  "Review these PRDs against the existing Reeves codebase.
   Are there patterns being violated? Existing code being duplicated?
   Is the scope appropriate or is it yak-shaving?"
```

Both run simultaneously. Results merged into a review doc at `.claude/ralphie-reviews/review.md`.

**Why two reviewers:** PAL catches technical issues. Second opinion catches strategic issues. Two perspectives > one.

### Phase 4: Build (Parallel Ralph Loops — 1 per workstream)

This is where ralphie kicks in. For each workstream:

1. Create branch: `ralphie/{task_id}-{slug}`
2. Generate prompt from PRD + review feedback
3. Launch `claude -p` with Ralph Loop iteration
4. Each agent builds on its own branch
5. Max iterations: 10 (enough for iteration, not enough to spin forever)

**All build agents run simultaneously.** Branch isolation prevents conflicts.

The prompt for each build agent includes:
- The PRD for its workstream
- Review feedback (gotchas to avoid)
- Acceptance criteria (how to know when done)
- File scope (what to touch, what to leave alone)

### Phase 5: Test & QA (Parallel)

As build agents complete, launch test agents:

**Agent: Test Runner**
```bash
# Per branch
git checkout ralphie/{task_id}-{slug}
.venv/bin/python -m pytest tests/ -q
# If failures, attempt fix (1 iteration)
```

**Agent: PAL Security Audit**
```
Use PAL's `secaudit` tool:
  Review the diff on branch ralphie/{task_id}-{slug}.
  Check for: XSS, injection, auth bypass, secrets in code.
```

**Agent: PAL Pre-commit Review**
```
Use PAL's `precommit` tool:
  Review all changes across all ralphie/* branches.
  Check for: code quality, naming conventions, dead code,
  missing error handling.
```

### Phase 6: Merge & Report

Once all agents complete:

1. **Merge each branch** into a staging branch `ralphie/merge`
2. **Run full test suite** on the merged result
3. **Generate summary** — what was built, what passed, what needs attention
4. **Devlog everything** — decisions, builds, reviews, test results
5. **Update Reeves tasks** — mark completed, note failures
6. **Notify** — push summary to devlog + optional Slack/notification

---

## Agent Roster & Tools

| Phase | Agent | Tool | Purpose |
|-------|-------|------|---------|
| 1. Decompose | PAL | `mcp__pal__planner` | Break vision into workstreams |
| 2. PRD | Claude | `claude -p` (x5-8) | Write specs per workstream |
| 3a. Review | PAL | `mcp__pal__codereview` | Technical review |
| 3b. Review | Janus | `janus_second_opinion` | Strategic review |
| 4. Build | Claude | `claude -p` + Ralph Loop (x5-8) | Implement features |
| 5a. Test | Claude | `claude -p` | Run tests, fix failures |
| 5b. Audit | PAL | `mcp__pal__secaudit` | Security review |
| 5c. Quality | PAL | `mcp__pal__precommit` | Code quality check |
| 6. Merge | Claude | `claude -p` | Merge, resolve, report |

**Total potential agents: 15-25 running across the pipeline.**

---

## Status & Monitoring

Status file: `.claude/maxralphie-status.json`

```json
{
  "vision": "Reeves 3.0 — Best-in-Class AI OS",
  "project_id": 4585,
  "started_at": "2026-02-14T03:00:00Z",
  "pipeline": {
    "decompose": { "status": "done", "workstreams": 6 },
    "prd": { "status": "running", "done": 3, "total": 6 },
    "review": { "status": "pending" },
    "build": { "status": "pending" },
    "test": { "status": "pending" },
    "merge": { "status": "pending" }
  },
  "agents": [
    { "phase": "prd", "task_id": 4370, "pid": 12345, "status": "running" },
    { "phase": "prd", "task_id": 4373, "pid": 12346, "status": "done" }
  ]
}
```

Check anytime:
```bash
cat .claude/maxralphie-status.json | jq '.pipeline'
```

---

## Why Each Tool Matters

**PAL (planner, codereview, secaudit, precommit):**
- Different model = different blind spots
- Structured review tools catch what Claude misses
- Security audit before merge prevents shipping vulnerabilities
- Pre-commit quality gate maintains codebase standards

**Janus (second_opinion):**
- Strategic layer — "should we even build this?"
- Catches scope creep and yak-shaving
- Cross-references with existing devlogs and patterns
- Different perspective than both Claude and PAL

**Ralph Loop (per build agent):**
- Iterative self-correction — first attempt is rarely perfect
- Each iteration sees previous work, improves on it
- 10 iterations enough for most features, prevents infinite spin
- Promise-based completion ensures genuine done-ness

**Reeves Integration:**
- Tasks pulled FROM Reeves (source of truth)
- Tasks marked complete IN Reeves (closes the loop)
- Devlog captures everything (durable learning)
- Project tracking stays current

---

## Example Invocations

```
/maxralphie "Reeves 3.0"
# Full pipeline: decompose → PRD → review → build → test → merge

/maxralphie "Add real-time collaboration to the knowledge graph"
# Free-text vision — PAL decomposes first, then full pipeline

/maxralphie "Reeves 3.0" --skip prd --skip review
# Just build and test (PRDs already written)

/maxralphie "Reeves 3.0" --max 3
# Conservative: only 3 concurrent agents

/maxralphie "Reeves 3.0" --dry-run
# Show the full pipeline plan without launching anything
```

---

## The Mental Model

You are the CEO. You describe the vision once.

Maxralphie runs your product org:
- **PAL** is your VP of Engineering — decomposes, reviews, audits
- **Janus** is your advisor — challenges assumptions, checks strategy
- **Ralph agents** are your engineers — build, iterate, ship
- **Reeves** is your project manager — tracks everything, closes loops
- **Devlog** is your company memory — nothing is lost

You walk away. Come back to: built features, reviewed code, passed tests, merged branches, updated tasks, comprehensive devlog.

That's not an AI tool. That's an AI company.
