# PAL MCP — Multi-Model Orchestration

## What PAL Is

PAL (Personal AI Layer) provides access to multiple AI models through MCP tools. It enables Reeves to get second opinions, architectural reviews, and consensus decisions from models with different training data and reasoning styles.

## Available Tools

| Tool | Purpose | Default Model |
|------|---------|---------------|
| `chat` | General conversation | Configurable |
| `planner` | Architecture decomposition, roadmap planning | GPT-5.2 |
| `codereview` | Technical code review | GPT-5.2 |
| `secaudit` | Security vulnerability scanning | GPT-5.2 |
| `precommit` | Code quality gate | GPT-5.2 |
| `debug` | Debugging assistance | GPT-5.2 |
| `testgen` | Test generation | GPT-5.2 |
| `docgen` | Documentation generation | GPT-5.2 |
| `analyze` | Code analysis | GPT-5.2 |
| `refactor` | Refactoring suggestions | GPT-5.2 |
| `challenge` | Devil's advocate on decisions | GPT-5.2 |
| `consensus` | Multiple models vote | Multi-model |
| `thinkdeep` | Extended reasoning | GPT-5.2 |
| `apilookup` | API documentation lookup | GPT-5.2 |
| `listmodels` | List available models | — |

## How PAL Is Used in Reeves 3.0

### In /mindreader

**PAL planner** synthesizes all gathered data:
```
"Given this person's data, what are the top priorities?
 What patterns do you see? What are they avoiding?"
```

**PAL consensus** validates the synthesis:
```
"Multiple models review the mindreader output.
 Do they agree on priorities? Flag disagreements."
```

### In /maxralphie

**Phase 0 — PAL planner** discovers what to build:
```
"Synthesize voice transcripts, devlogs, tasks, brain suggestions,
 and sleep briefings into a prioritized product roadmap."
```

**Phase 1 — PAL planner** decomposes vision:
```
"Break this vision into 5-8 independent workstreams.
 For each: name, goal, files, dependencies, acceptance criteria."
```

**Phase 3 — PAL codereview** reviews PRDs:
```
"Review these PRDs for conflicts, missing edge cases,
 security concerns, performance gotchas."
```

**Phase 3 — PAL challenge** questions assumptions:
```
"Are we building the right things? Is this scope appropriate?
 Are there patterns being violated?"
```

**Phase 5 — PAL secaudit** checks security:
```
"Review diffs for XSS, injection, auth bypass, secrets in code."
```

**Phase 5 — PAL precommit** checks quality:
```
"Review changes for code quality, naming conventions,
 dead code, missing error handling."
```

### In /ralphie

PAL is not directly used in ralphie (it's focused on execution speed). But PAL's review outputs from maxralphie inform the ralphie agent prompts.

## Why Multi-Model Matters

**Different training data = different blind spots.**

| Model | Strength | Blind Spot |
|-------|----------|-----------|
| Claude (Opus 4.6) | Nuanced reasoning, long-context | May not catch its own patterns |
| GPT-5.2 | Structured analysis, security | Different code conventions |
| Gemini | Broad knowledge, search | Less code generation depth |
| O3 | Mathematical reasoning | Less creative synthesis |

**PAL consensus** is especially valuable for architecture decisions — when 3+ models agree, confidence is high. When they disagree, it surfaces a genuine trade-off worth discussing.

## Integration Pattern

```python
# In a slash command prompt:
# 1. Gather data from Reeves MCP
# 2. Feed to PAL for synthesis
# 3. Use PAL's output to guide Claude's implementation
# 4. PAL reviews Claude's work before merge
```

The pattern is: **Claude does, PAL reviews.** Or: **PAL plans, Claude builds.**

This creates a check-and-balance that catches more issues than any single model alone.

## Model Selection

When a user specifies a model, PAL uses it exactly. When no model is specified, use `listmodels` to discover available models and select the best fit.

For Reeves 3.0, the default workflow uses:
- **GPT-5.2** for structured review tasks (codereview, secaudit, planner)
- **Consensus (multi-model)** for high-stakes decisions (architecture, strategy)
- **Claude Opus 4.6** for implementation (code generation, long synthesis)
