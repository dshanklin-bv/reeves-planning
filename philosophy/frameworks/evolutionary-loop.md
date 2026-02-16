# The Evolutionary Loop

## The Six Steps

```
OBSERVE   ->  Sense the environment (passive collection)
MODEL     ->  Build a representation (emotional + structural + semantic)
INFER     ->  Predict what's needed (from gaps between words and actions)
COLLAPSE  ->  Act to close the distance (maximum system work, minimum human work)
GOVERN    ->  Ensure safety (structural testing, not permission dialogs)
LEARN     ->  Feed outcomes back (reinforce or decay, the model improves)
```

This is the loop that biology runs. It's the loop that Reeves implements. It's the loop that current AI is missing.

## Biological Mapping

| Step | Biology | Timescale |
|------|---------|-----------|
| OBSERVE | Senses detect environment (eyes, ears, skin, gut) | Milliseconds |
| MODEL | Brain builds world model (neural patterns, predictions) | Ongoing |
| INFER | Predict what's needed (hunger before starvation, fear before danger) | Milliseconds-hours |
| COLLAPSE | Act (eat, flee, build, communicate) | Seconds-hours |
| GOVERN | Immune system, pain signals, social norms prevent harmful actions | Ongoing |
| LEARN | Neural plasticity, immune memory, behavioral conditioning | Hours-lifetime |

At a longer timescale, evolution runs the same loop across generations:

| Step | Evolution | Timescale |
|------|-----------|-----------|
| OBSERVE | Organisms interact with environment | Lifetime |
| MODEL | Genome encodes environmental adaptations | Generational |
| INFER | Encoded heuristics predict threats/opportunities | Instantaneous (instinct) |
| COLLAPSE | Organism acts on instinct + learning | Lifetime |
| GOVERN | Lethal failures are removed from gene pool | Generational |
| LEARN | Successful adaptations persist in genome | Generational |

## What Makes the Loop Work

### 1. It's Continuous
The loop doesn't run once. It runs constantly. Every observation feeds the model. Every action produces an outcome. Every outcome updates the model. There is no "training phase" and "deployment phase." There is only the loop.

### 2. Selection Pressure Is Real
In biology, the consequences of bad actions are real — injury, death, social exclusion, starvation. This harshness is what makes the loop produce dense, useful information. Gentle feedback produces gentle improvements. Harsh feedback produces survival-grade heuristics.

### 3. Information Compounds
Each cycle doesn't start fresh. It builds on every previous cycle. The immune system remembers pathogens. The brain remembers experiences. DNA remembers successful adaptations. Information accumulates and gets denser over time.

### 4. Multiple Timescales
Biology runs the loop at multiple timescales simultaneously:
- **Milliseconds:** Neural firing (reflexes, perception)
- **Hours:** Learning (skill acquisition, habituation)
- **Years:** Development (brain maturation, personality)
- **Generations:** Evolution (genetic adaptation)

Each timescale feeds the others. Short-term learning shapes long-term development. Long-term evolution shapes the capacity for short-term learning.

## The AI Implementation

### OBSERVE
Passive collection from every surface a person's life touches:
- Voice conversations (what they say)
- Task behavior (what they do and avoid)
- Financial transactions (how they spend)
- Messages (who they talk to)
- Calendar (how they allocate time)
- Browser/UI telemetry (what they check obsessively)

### MODEL
Build a multi-layered representation:
- **PEFM** (emotional weighting — urgency, insight, frustration, satisfaction, complexity, novelty)
- **Knowledge graph** (relationships between entities, confidence-scored)
- **Embeddings** (semantic similarity across all content)
- **Intents + Facts** (stated goals, biographical data)

### INFER
Find the gaps between what the person says and what they do:
- Mentioned 4 times, zero actions taken = procrastination
- Relationship with decreasing contact = drift
- Recurring charges never reviewed = financial blind spot
- High PEFM urgency + zero task progress = avoidance

### COLLAPSE
Do maximum work so the person does minimum:
- Break stalled tasks into subtasks with the first one taking 10 minutes
- Draft messages for relationship maintenance
- Pre-fill forms, schedule calendar blocks
- Navigate to the right page, pre-load context

### GOVERN
Structural safety, not permission dialogs:
- Brain module evaluates actions against constraints
- 939+ automated tests prevent regressions
- Human review gates for financial/relationship/external actions
- Soft deletes only — nothing permanently destroyed
- Multiple AI models review high-stakes decisions

### LEARN
Feed outcomes back into the model:
- Did the person complete the task? Reinforce the approach
- Did they ignore it? Learn — maybe the barrier was anxiety, not time
- PEFM dimensions decay at different rates (urgency fades fast, insight persists)
- Sleep cycles extract cross-domain patterns
- Pattern boost amplifies confirmed heuristics

## This Is Lamarckian Evolution

There's a fair objection: "This isn't really evolution. Where's the population? Where's differential survival? Where's natural selection?"

The objection assumes Darwinian evolution — random mutation, blind selection, information passed only through the genome. And it's right that Reeves doesn't do that. What Reeves does is more powerful.

Reeves implements **Lamarckian evolution** — the organism acquires traits during its lifetime and passes them directly to the next generation. In biology, this doesn't happen. What you learn doesn't rewrite your DNA. Your children start from the same genome regardless of what you figured out. That's why biological evolution is slow — every generation rediscovers from scratch, guided only by which organisms survived.

But Reeves' spec *is* rewritten by what the system learns. Each Ralph Loop cycle produces a new generation where acquired knowledge — patterns that worked, patterns that failed, heuristics discovered at runtime — gets encoded directly into the spec that builds the next version. The spec is the genome, and unlike DNA, it absorbs lived experience.

This is exactly what cultural evolution does, and it's why human civilization advanced faster than the human genome. Writing, institutions, science — these are Lamarckian inheritance mechanisms. What one generation learns, the next generation starts from. Biology took 3.8 billion years to produce a brain. Cultural evolution took that brain from campfires to quantum computers in 50,000 years. Same principle: acquired traits passing forward.

The mapping is precise:

| Evolutionary Component | Darwinian Biology | Reeves (Lamarckian) |
|----------------------|-------------------|---------------------|
| **Genome** | DNA | The spec |
| **Generation** | Birth to reproduction | One Ralph Loop cycle |
| **Variation** | Random mutation | Each rebuild interprets the spec against current environment |
| **Selection** | Differential survival | Real data — failed patterns drop, successful ones persist |
| **Inheritance** | Genome passes to offspring (unchanged by life) | Spec passes to next cycle (updated with learned patterns) |
| **Death** | Organism dies | Old version replaced — code doesn't survive except through what the spec inherited |
| **Cycle time** | Years to decades | Hours |

That last row is the killer advantage. Biological evolution is gated by generation time — 25 years for humans, 20 minutes for bacteria. Reeves runs a full evolutionary cycle in hours. Compressed cycle times with Lamarckian inheritance means the system evolves at a rate biology cannot touch.

This isn't a weaker form of evolution. It's a stronger one. Biology couldn't do Lamarckian inheritance, so it settled for the slow version. AI doesn't have that constraint.

## The Key Difference from Current AI

Current AI: Train → Deploy → Serve (static)
The loop: Observe → Model → Infer → Collapse → Govern → Learn → Observe → ... (continuous)

In current AI, the model that serves user #1,000,000,000 is identical to the one that served user #1. A billion conversations happen. The model learns nothing from any of them.

In the evolutionary loop, every cycle changes the system. The model that runs tomorrow is different from the one that runs today — not because someone retrained it, but because it *lived* another day and accumulated more experience. That's the difference between a static tool and an evolving system.
