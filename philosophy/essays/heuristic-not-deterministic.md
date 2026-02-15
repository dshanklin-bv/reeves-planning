# Heuristic, Not Deterministic

*Why evolution beats engineering for open-ended problems — and what that means for AI.*

---

## Two Kinds of Systems

**Deterministic systems** handle scenarios they were designed for. Given input X, produce output Y. The mapping is explicit, precise, and reliable — within the designed domain. Outside that domain, the system fails, usually silently.

**Heuristic systems** handle scenarios that have never existed before. They use approximate rules — "good enough" strategies discovered through experience — that generalize far beyond their origin. They're imprecise by design. That imprecision is their power.

Engineering builds deterministic systems. Evolution builds heuristic ones.

## Biology's Heuristics

**The immune system.** There is no database of pathogens in your body. Instead, your immune system runs a heuristic process: generate random antibodies through VDJ recombination, test each one against the invader, massively reproduce the ones that bind, kill the rest. This process handles viruses that have *never existed in the history of life on Earth*. It works on first encounter. Not because the solution was pre-programmed, but because the heuristic for *finding solutions* was evolved.

**Fear responses.** Your fear of heights isn't a list of dangerous places. It's an approximate rule: high + edge = danger. This fires on cliffs, rooftops, glass floors, VR environments, airplane windows, roller coasters — scenarios no ancestor specifically encountered. The heuristic is "wrong" in many cases (glass floors are safe) but it keeps you alive in all the cases that matter. That's the trade-off: precision for coverage.

**Pattern recognition.** You can recognize a face you've seen once, at a different angle, in different lighting, years later. No lookup table. A heuristic pattern matcher that trades exact recall for generalization across infinite visual conditions.

**Metabolism.** Your body maintains blood glucose within a narrow range whether you eat a steak, a candy bar, or nothing at all. It handles foods it has never encountered by applying heuristic metabolic strategies evolved over millions of years of variable diet.

## Why This Matters for AI

Current AI is trained deterministically. Gradient descent on a fixed dataset produces weights that are a deterministic function of the training data. The same input produces the same output (modulo temperature sampling). This is enormously capable within the training distribution.

But the real world is not the training distribution.

When a person uses an AI system, they bring novel situations — combinations of context, emotion, history, and need that no training example precisely matches. The AI's response is an interpolation across its training data. Sometimes brilliant. Sometimes a hallucination — a confident-sounding answer fabricated because the system has no heuristic for "I've never seen this before."

A heuristic system handles novelty differently. It doesn't interpolate from training examples. It applies approximate strategies and observes the result. Did it work? Reinforce. Didn't work? Adjust. Over time, the strategies get shaped by reality rather than by a static dataset.

## The Infinity Problem

Here's why this distinction is fatal for the "just scale it" approach:

The space of scenarios a person will encounter in their life is effectively infinite. You can't enumerate them. Every day brings novel combinations of context, need, and constraint that have never occurred in exactly that configuration before.

**Deterministic approach:** Handle more cases. More data, more parameters, more fine-tuning. Each new case = more storage. This scales linearly and never finishes because the case space is infinite.

**Heuristic approach:** Discover approximate strategies that cover categories of cases. One heuristic covers an infinite number of specific scenarios. "High + edge = danger" doesn't enumerate cliffs. It covers all of them with a single rule.

This is why DNA is 750 MB and AI models are terabytes. Heuristics compress infinitely better than lookup tables. They don't store answers to specific problems. They store *strategies for classes of problems*.

## How Heuristics Are Discovered

This is the critical question. You can't engineer a heuristic the way you engineer a deterministic rule. Heuristics are **discovered through selection pressure**, not designed through analysis.

The fear-of-heights heuristic wasn't designed by a biologist. It emerged because primates without it fell and died. Over millions of generations, the approximate rule "high + edge = danger" proved to be the strategy with the best survival rate. It's not optimal. It's not precise. It's *fit* — the strategy that survived the most selection pressure.

To discover heuristics, you need:
1. **Variation** — Many different strategies attempted
2. **Selection** — Real consequences for strategies that don't work
3. **Inheritance** — Successful strategies persist and compound
4. **Time** — Enough cycles for approximate rules to crystallize

This is, definitionally, evolution. Not metaphorical evolution. The actual mechanism: variation, selection, inheritance, time. It's the only known process that reliably produces heuristic systems capable of handling open-ended environments.

## The Claim

AI will not reach its full potential through deterministic engineering alone.

Not because engineering is bad. Engineering built the substrate — the transformer, the attention mechanism, the scaling laws. That substrate is extraordinary.

But deterministic systems don't scale to non-deterministic environments. And the real world — a person's life, with all its novelty, ambiguity, and surprise — is the most non-deterministic environment there is.

What's needed is an evolutionary loop: a system that tries things in the real world, experiences real consequences, remembers what worked, and passes that knowledge forward. Over enough cycles, this loop will discover heuristics — approximate strategies that handle scenarios no engineer anticipated — because that's what evolutionary loops do. It's the only thing they do. And given enough time, it's enough for everything.
