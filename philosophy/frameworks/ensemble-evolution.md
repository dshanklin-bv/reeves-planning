# Ensemble Evolution: The Pattern for Iterative Improvement

*Randomized critics with different loss functions are artificial selection applied to design.*

---

## The Problem

A builder improving their own work converges to a local maximum. They optimize for what they already value. They can't see what they can't see.

This is the same trap biology avoids through sexual reproduction: if organisms only replicated themselves, variation would collapse. Novel environments would kill the population. The power of sexual reproduction is recombination — mixing genetic material from different sources to explore more of the fitness landscape.

## The Pattern

**Ensemble Evolution** applies this to any creative or engineering process:

1. **Define critics with different loss functions** — each one optimizes for something different and kills something different
2. **Select randomly** — prevents the builder from cherry-picking friendly reviewers
3. **Get exactly ONE critique per cycle** — not a laundry list. The single highest-impact intervention from that critic's lens
4. **Implement ONE fix** — isolate variables. Can't tell what helped if you change five things
5. **Capture evidence** — screenshots, metrics, diffs. If it's not recorded, it didn't happen
6. **Update the documentation** — the process document IS the institutional memory. It evolves alongside the artifact

### Why Random Selection

The critics are loss functions. A Design Lead minimizes visual clutter. A Growth Lead minimizes time-to-comprehension. A Copywriter minimizes wasted words. An Essay Editor minimizes reading friction.

If you always pick the same critic, you converge to their optimum — which might be a terrible solution for a different lens. Random selection ensures the full fitness landscape is explored over enough iterations.

This is the same reason evolution works: **the environment varies**. Organisms that survive drought die in flood. Organisms that thrive in heat die in cold. The randomness of environmental challenge is what produces generalists capable of handling anything.

Randomized critics = randomized environmental challenge for your design.

### Why One Change Per Cycle

Multi-variable changes make attribution impossible. If you change the typography AND the layout AND the color palette in one iteration, and the result is better, you don't know which change helped. If the result is worse, you don't know which change hurt.

Biology isolates variables through **point mutations** — small, targeted changes that can be individually selected for or against. The evolve pattern does the same: one structural change, evaluated, kept or discarded.

### Why External Reviewers

The builder is biased. They made the thing. They see what they intended, not what exists. An external reviewer sees what's actually there — with fresh eyes and a different optimization target.

This is why peer review exists in science, code review exists in engineering, and editors exist in publishing. The person who wrote it is the worst judge of whether it works. Not because they're bad — because they're too close.

AI reviewers (PAL, GPT, etc.) are useful here because they're cheap, fast, available, and can be given arbitrary personas. You can't hire 8 expert reviewers for a personal site. You can simulate them.

### Why Self-Updating Documentation

The skill file that describes the process IS the institutional memory. It records:
- What was tried (iteration log)
- What worked (feature locks)
- What failed (anti-patterns)
- What the current state is (architecture notes)

Without this, each session starts from zero. With this, each session builds on everything that came before. The skill file is the DNA of the process — it persists what was learned across generations (sessions).

## The Evolutionary Frame

| Concept | Biology | Ensemble Evolution |
|---------|---------|-------------------|
| Variation | Mutation + recombination | Random persona selection |
| Selection | Environmental pressure | Critic feedback (one structural change) |
| Inheritance | DNA passing to offspring | Skill file persisting across sessions |
| Time | Generations | Iterations |
| Fitness | Survival + reproduction | Scores from fresh-eyed critics |
| Feature lock | Highly conserved genes | Aspects that score 8+ for 2+ rounds |
| Local maximum | Isolated population | Single-reviewer convergence |
| Fitness landscape exploration | Sexual reproduction / migration | Random critic rotation |

## The Kill Shot

> A single reviewer tells you how to make it better for them.
> Random reviewers tell you how to make it better for everyone.
> The randomness IS the selection pressure.

## Where This Is Used

| Implementation | Domain | Personas | Evidence |
|----------------|--------|----------|----------|
| zeus-evolve | SaaS demo page | 11 critics (Design Lead → Behavioral Psych) | Playwright screenshots |
| render-loop | Blender wallpaper | 3 critics (Art Director, Graphics Prog, Photographer) | Render PNGs |
| ds-dot-com evolve | Personal philosophy site | 8 critics (Essay Editor → Info Architect) | Playwright screenshots |

The pattern is the same. Only the domain, personas, and evidence format change.

## Connection to Other Frameworks

**Search and Graph**: Universal search is the sensory intake; the knowledge graph is memory consolidation. Ensemble Evolution is the selection pressure that decides what's worth keeping — the promotion loop applied to design decisions rather than knowledge.

**The Realistic Timeline**: This is Era 1 (Code) technique. Humans are still the selection pressure. But the ensemble of AI critics is a bridge to Era 2 — where the specs (personas + critique templates) begin to drive the evolution with less human intervention per cycle.

**Heuristic, Not Deterministic**: The persona prompts are heuristics. "You HATE clutter" is not a precise rule — it's an approximate strategy that generalizes across infinite specific design decisions. The personas are evolved heuristics for catching design problems.
