# Claude Opus 4.6 — Capabilities for Reeves 3.0

## Why Opus 4.6 Matters

Opus 4.6 is the most capable model available for autonomous coding and complex reasoning. Reeves 3.0's architecture depends on capabilities that only exist at this tier.

## Key Specs

| Capability | Value | Why It Matters for Reeves |
|-----------|-------|---------------------------|
| Context Window | 1M tokens | Load entire codebase + conversation history + data |
| Output Tokens | 128K | Generate complete PRDs, vision docs, full implementations |
| Terminal-Bench | 65.4% | Best autonomous coding score — ralph loops work |
| Extended Thinking | Yes | Hard architectural decisions get proper reasoning |
| Tool Use | Advanced | 100+ MCP tools orchestrated reliably |
| Parallel Subagents | 10 | Multiple ralph agents running simultaneously |
| Self-Correction | Yes | Sees own mistakes, iterates without human intervention |

## Capabilities That Enable Reeves 3.0

### 1. Million-Token Context
- Load `voice.py` (574 lines) + `brain.py` (898 lines) + `pefm.py` (466 lines) + entire test suite in one context
- Cross-reference voice transcripts with task data with finance data
- Mindreader's god-depth synthesis requires holding all data sources simultaneously

### 2. 128K Output Tokens
- Complete PRD generation in a single pass
- Full vision document synthesis (mindreader `--act` output)
- Long code generation for new features
- No truncation on comprehensive analysis

### 3. Self-Correcting Loops (Ralph Loop)
- Agent sees its own previous work in files
- Identifies mistakes, iterates, improves
- 65.4% Terminal-Bench = reliable autonomous coding
- 10 iterations enough for most features

### 4. Extended Thinking
- Architecture decisions (which pillar first?)
- Cross-domain pattern analysis (sleep cycle reasoning)
- Trade-off evaluation (speed vs safety in governance)
- Not just fast — genuinely thinking through hard problems

### 5. Parallel Subagents (10 concurrent)
- ralphie spawns 5-10 agents building features simultaneously
- maxralphie Phase 2: 5-8 agents writing PRDs in parallel
- mindreader: 8 data gathering agents running simultaneously
- Massive throughput improvement over sequential execution

### 6. Tool Orchestration
- 100+ MCP tools available (Reeves, PAL, Tosh, GitHub, Outlook, etc.)
- Reliable selection of appropriate tool for each step
- Chain tools across domains (query finance → check calendar → draft message)
- No hallucinated tool names or parameters

## What Opus 4.6 Can Do That Others Can't

| Scenario | Opus 4.6 | Smaller Models |
|----------|----------|----------------|
| Read entire reeves-web codebase | Yes (fits in 1M context) | Truncated or windowed |
| Generate full PRD + implementation | Single pass | Multiple passes with drift |
| Self-correct across 10 iterations | Reliable improvement each round | Regressions common |
| Orchestrate 25 tool calls in a session | Maintains coherence | Loses track after ~10 |
| Cross-domain synthesis (voice + finance + graph) | Unified reasoning | Siloed analysis |
| Extended thinking on architecture | Deep structured reasoning | Surface-level responses |

## Deployment in Reeves

- **Primary engine**: Claude Code CLI (`claude -p` for headless, interactive for dev)
- **Model ID**: `claude-opus-4-6`
- **Subagent model**: Inherits from parent (stays Opus 4.6)
- **Cost consideration**: Opus is expensive — use for complex tasks, not routine queries
- **Fast mode**: Same model, faster output (toggle with `/fast`)

## Integration with PAL

Opus 4.6 + PAL (GPT-5.2) provides multi-model reasoning:
- Claude: Primary synthesis, code generation, long-context reasoning
- PAL: Second perspective, security audit, architectural challenge
- Consensus tool: Multiple models vote on approach
- Different training data = different blind spots caught
