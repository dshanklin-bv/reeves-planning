# Development Methodology

How we build things with AI agents. Not the Reeves philosophy (WHY we build) — that's in `philosophy/`. This is the planning philosophy: HOW to decompose work, when to spec vs. ship, and how to let architecture emerge from iteration.

## Core Principle

**AI can brute-force its way to the right architecture through rapid iteration — but only with the right decomposition strategy.** The methodology is about choosing the right strategy for the problem shape.

## Two Decomposition Strategies

### Elephant Conga Carpaccio — for Pipelines

When the dependency chain is known and the steps can't be reordered.

```
🐘→🐘→🐘→🐘→🐘
```

Each elephant is a capability. The conga order is the dependency order. Each elephant gets sliced into the thinnest vertical deliverables (carpaccio). Run real data after every slice.

**Use when:** You know the destination and the dependencies are sequential. Source accounts → payee normalization → merchant discovery → categorization → enrichment. You can't skip ahead.

**The tell:** If reordering the steps would break things, it's a conga.

See: `elephant-conga-carpaccio.md`

### Skateboard → Bike → Car — for Products

When the destination is uncertain and each version must deliver standalone value.

```
🛹 → 🚲 → 🏍 → 🚗
```

Each version is a complete product that solves the core problem at increasing fidelity. You might learn from the skateboard that nobody wants a car — they want a motorcycle. Each version teaches you what the next version should be.

**Use when:** You're exploring what to build, not how to build it. A new service, a new UI, a new interaction model. The first version might change the plan entirely.

**The tell:** If you could skip or reorder the versions and each still makes sense on its own, it's skateboard-bike-car.

### When to Use Which

| Situation | Strategy | Why |
|-----------|----------|-----|
| Data pipeline (ingest → transform → load) | Conga | Steps are sequential dependencies |
| New service (router, mechanic) | Skateboard | Ship simplest useful version, learn |
| Cross-service integration (finance ↔ email) | Conga | Each integration enables the next |
| New UI/dashboard | Skateboard | Users tell you what's wrong with v1 |
| Schema/entity model | Conga | Later tables depend on earlier ones |
| AI feature (categorization, enrichment) | Conga | Each signal source depends on cleaner data from the previous |
| Entire Reeves vision | Skateboard | What does v1 of a mind-reading OS look like? |
| Single service's maturation | Both | Skateboard for the service itself, conga for its internal pipeline |

### They Compose

A product (skateboard) can contain a pipeline (conga). The skateboard version of reeves-finance is "transactions come in, entries come out." Inside that skateboard, the pipeline is a conga: source mapping → normalization → categorization → validation. The skateboard decides WHAT to build. The conga decides HOW to sequence the internals.

## Spec-Driven Development

Write executable specs before code. Specs define behavioral scenarios, acceptance criteria, commands, and boundaries. Agents pick them up cold and execute.

**Specs are the conga choreography.** They define the elephants, their order, and the acceptance criteria for each slice. Without a spec, an agent doing a Ralph Loop session has no direction. Without carpaccio slices in the spec, the agent builds thick horizontal layers instead of thin vertical ones.

Specs live in `../specs/`.

### Spec Lifecycle
```
Spec → Slice → Build → Run Real Data → Learn → Update Spec → Next Slice
```

The spec is a living document. Every slice that runs against real data might reveal that the spec was wrong. Update it. The spec captures what you've learned, not what you guessed upfront.

## Meta-Cognition

The methodology is recursive: it evaluates itself. See `meta-cognition.md` for the full framework:

- **Plan → Act → Reflect → Refine** loop (explicit, not silent)
- **Per-slice meta-checklist** (thickness audit, real data check, strategy check, PAL engagement)
- **PAL as meta-agent** (consult before large changes, after failures, for multi-model analysis)
- **Thinking journal** (git history + `meta/` notes track the evolution of plans, not just code)

## The Three Rules

1. **Run real data after every slice.** The slice isn't done when the code compiles. It's done when you run it against reality and see what it does.

2. **The output of each slice informs the next.** Don't plan 10 slices upfront. Plan 1, run it, see what you learn, plan the next one.

3. **Architecture is emergent, not designed.** You discover the right abstractions by building the simplest thing that works, running it, and seeing what breaks.
