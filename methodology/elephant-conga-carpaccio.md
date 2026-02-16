# Elephant Conga Carpaccio

A development framework for AI-driven systems where capabilities form a dependency chain and each capability is sliced into the thinnest possible vertical deliverables.

## The Metaphor

Elephants in a conga line, trunk to tail. Each elephant is a capability. The line order is the dependency order — you can't move elephant 4 until elephant 3 steps forward. And each elephant gets sliced into carpaccio: the thinnest vertical cuts that still deliver end-to-end value.

```
🐘→🐘→🐘→🐘→🐘→🐘
```

## Three Rules

1. **The conga order is fixed.** Dependencies determine sequence. Don't skip ahead because a later elephant is more exciting. The chain breaks.

2. **Each elephant gets carpaccio'd independently.** Before moving to the next elephant, slice the current one thin enough that it's standing — working, tested, delivering value. You don't need to finish it. You need it weight-bearing.

3. **Run real data after every slice.** The slice isn't done when the code compiles. It's done when you run it against reality and see what it does. The output of each slice informs the next slice — and sometimes reorders the conga.

## Why This Works for AI Development

Traditional software has high shipping overhead — CI, review, deploy, rollback. So teams batch work into thick slices to amortize the overhead. The result: late feedback, wrong assumptions baked in, expensive rework.

AI agents don't have that overhead. An agent can ship 10 slices in the time a human ships 1. Each slice runs against real data immediately. The feedback loop is minutes, not days.

This changes the optimal strategy: **don't plan the architecture, iterate into it.** The conga provides direction (you know the dependency order). The carpaccio provides speed (each step is tiny and testable). Real data provides truth (the output corrects your assumptions).

Architecture becomes emergent rather than designed. You discover the right abstractions by building the simplest thing that works, running it, and seeing what breaks.

## Example: Autonomous Bookkeeping

The goal: AI does the books. The conga:

```
🐘 Source Account Map
  → fixes debit/credit side of every transaction
  → can't categorize correctly until this works

🐘 Payee Normalizer
  → cleans dirty bank strings into matchable names
  → can't match merchants against "Tst* Palm Wood - Brafort Worth TX"

🐘 Merchant Discovery
  → auto-creates merchant entities from normalized payees
  → can't do merchant→account mapping without merchants

🐘 Categorization Pipeline
  → merchants + rules + ACCOUNT_HINTS → confident categorization
  → can't know what to enrich until you know what's uncertain

🐘 Email Enrichment
  → receipts boost confidence for uncertain transactions
  → try the cheap signal before the expensive one

🐘 LLM Fallback
  → claude -p for truly unknown transactions
  → last resort, most expensive, but learns rules for next time
```

Each elephant gets 2-4 carpaccio slices. Each slice ends with `reeves-finance auto --limit N --json` against real data. The output tells you whether to keep slicing the current elephant or step forward to the next one.

## When the Conga Reorders

Sometimes running real data reveals that your dependency order was wrong. You thought you needed merchants before categorization, but it turns out the payee normalizer is so effective that regex rules catch 80% of transactions. Merchant discovery drops from elephant 3 to elephant 5.

That's fine. The conga isn't a spec — it's a hypothesis about dependency order. Reality corrects it. The important thing is that you have a conga (direction) and you're running real data (feedback). The order will sort itself.

## Relationship to Other Frameworks

- **Elephant Carpaccio** (Cockburn): The slicing principle. Conga adds the dependency chain.
- **Story Mapping** (Patton): Horizontal journey, vertical slices. Conga is more opinionated about ordering.
- **Walking Skeleton** (Cockburn): Thinnest end-to-end path. Conga is a chain of walking skeletons, each one enabling the next.
- **Spec-Driven Development**: Specs define the elephants and their order. Carpaccio slices are the acceptance criteria within each elephant. The spec is the conga choreography.

## The Mantra

Identify the conga. Slice the first elephant. Run real data. Repeat.
