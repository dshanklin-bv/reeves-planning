# Elephant Carpaccio

Alistair Cockburn's principle: slice work into the thinnest possible vertical slices that still deliver end-to-end value. Each slice is complete, testable, deployable, and meaningful on its own.

## Why This Matters for Reeves

AI can brute-force its way to the right architecture through rapid iteration — but only if each iteration is small enough to evaluate, cheap enough to throw away, and complete enough to learn from. Big horizontal layers ("build the whole data model, then the whole API, then the whole UI") don't give feedback until the end. Thin vertical slices give feedback immediately.

## The Rule

Every increment must be:
- **Vertical** — touches every layer needed (DB, logic, CLI/API, test) but only for one narrow behavior
- **Complete** — works end-to-end without requiring the next increment
- **Testable** — has acceptance criteria that can be verified in under a minute
- **Reversible** — if it's wrong, rip it out without collateral damage

## What This Looks Like in Practice

**Wrong (horizontal layers):**
```
Week 1: Build all database tables for finance, email, health, relationships
Week 2: Build all API endpoints
Week 3: Build all CLI commands
Week 4: Wire everything together and pray
```

**Right (elephant carpaccio):**
```
Slice 1: One transaction → categorized → journal entry → validated → stored (2 hours)
Slice 2: Batch process N transactions with same pipeline (1 hour)
Slice 3: Add confidence threshold → auto-approve vs pending (1 hour)
Slice 4: Add rule learning from approved entries (2 hours)
Slice 5: Schedule the pipeline to run nightly (1 hour)
```

Each slice is shippable. Each one teaches you something about the next one. By slice 5, you've built the same system as the horizontal approach — but you had working software from slice 1.

## Application to Specs

Every spec in `reeves-planning/specs/` must decompose into slices, not phases. The difference:

- **Phase** = "build the enrichment system" (weeks of work, no feedback until done)
- **Slice** = "enrich one transaction with one email receipt lookup" (hours of work, immediate feedback)

When writing a spec, ask: "What's the thinnest slice that proves this works?" Build that first. Then ask again.

## The AI Advantage

Humans resist thin slices because the overhead of shipping feels high relative to the work. AI doesn't have that problem. An agent can ship 10 thin slices in the time a human ships 1 thick phase — and each slice course-corrects the next one. The iteration speed makes the architecture emergent rather than planned.

This is why spec-driven development + elephant carpaccio + AI agents is a fundamentally different development model. The spec provides direction. The slices provide feedback. The AI provides speed. The architecture falls out of the iteration.

## See Also

**Elephant Conga Carpaccio** (`elephant-conga-carpaccio.md`) extends this with dependency chains. When capabilities depend on each other, the conga line defines the order. Each elephant gets carpaccio'd independently.

**Skateboard → Bike → Car** is the alternative strategy for products where the destination is uncertain. See `README.md` in this directory for when to use which.
