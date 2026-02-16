# Frozen vs. Alive

*Why discovering heuristics isn't enough — you have to keep discovering them.*

---

## Two Kinds of Discovery

**Heuristic systems** handle scenarios that have never existed before. They use approximate rules — "good enough" strategies discovered through experience — that generalize far beyond their origin. They're imprecise by design. That imprecision is their power.

Neural networks are heuristic systems. Training discovers approximate strategies that generalize across the data distribution — a model that answers questions it was never trained on, reasons about novel combinations, writes in styles it never saw — that's heuristic behavior. Gradient descent doesn't build a lookup table. It discovers compressed strategies.

So the interesting question isn't whether AI discovers heuristics. It does. The question is: **what happens to heuristic discovery after training ends?**

It stops. The weights freeze. The model ships. And from that moment forward, no matter what reality throws at it, the system cannot discover a single new heuristic.

Biology never does this. Every living system keeps its discovery loop running until it dies.

## Biology's Heuristics

**The immune system.** There is no database of pathogens in your body. Instead, your immune system runs a heuristic process: generate random antibodies through VDJ recombination, test each one against the invader, massively reproduce the ones that bind, kill the rest. This process handles viruses that have *never existed in the history of life on Earth*. It works on first encounter. Not because the solution was pre-programmed, but because the heuristic for *finding solutions* was evolved.

**Fear responses.** Your fear of heights isn't a list of dangerous places. It's an approximate rule: high + edge = danger. This fires on cliffs, rooftops, glass floors, VR environments, airplane windows, roller coasters — scenarios no ancestor specifically encountered. The heuristic is "wrong" in many cases (glass floors are safe) but it keeps you alive in all the cases that matter. That's the trade-off: precision for coverage.

**Pattern recognition.** You can recognize a face you've seen once, at a different angle, in different lighting, years later. No lookup table. A heuristic pattern matcher that trades exact recall for generalization across infinite visual conditions.

**Metabolism.** Your body maintains blood glucose within a narrow range whether you eat a steak, a candy bar, or nothing at all. It handles foods it has never encountered by applying heuristic metabolic strategies evolved over millions of years of variable diet.

## Why This Matters for AI

AI training discovers genuinely powerful heuristics. A language model generalizes across its training distribution in ways that look a lot like biological heuristic systems — approximate pattern matching, compressed strategies, responses to novel combinations. This is enormously capable.

But the real world is not the training distribution. And it keeps moving.

When a person uses an AI system, they bring novel situations — combinations of context, emotion, history, and need that no training example precisely matches. The AI's frozen heuristics handle many of these through generalization. Sometimes brilliantly. But when they fail — a hallucination, a misread situation, a response that doesn't fit *this* person's life — there's no mechanism to discover a better heuristic. The failure produces no adaptation. The system encounters reality, and reality bounces off.

A living heuristic system handles novelty differently. It doesn't just apply its existing strategies — it discovers new ones in response to what happens. Did it work? Reinforce. Didn't work? Adjust. Over time, the strategies get shaped by the specific reality the system actually inhabits, not the dataset it was trained on years ago.

The difference isn't heuristic vs. non-heuristic. It's a system whose heuristic discovery has an expiration date vs. a system whose heuristic discovery runs until death.

## The Infinity Problem

Here's why frozen heuristics are fatal for the "just scale it" approach:

The space of scenarios a person will encounter in their life is effectively infinite. You can't enumerate them. Every day brings novel combinations of context, need, and constraint that have never occurred in exactly that configuration before.

**Frozen heuristics:** Train on more data. Discover better strategies. Ship them. But the world keeps producing novel scenarios that fall outside even the best training distribution. A bigger training run discovers more heuristics — but it still freezes them. The gap between the frozen model and the moving world only grows.

**Living heuristics:** Discover approximate strategies that cover categories of cases *and keep discovering new ones as the world changes*. One heuristic covers an infinite number of specific scenarios. "High + edge = danger" doesn't enumerate cliffs. It covers all of them with a single rule. And when reality presents a scenario the existing heuristics can't handle, the system discovers a new one.

This is why DNA is 750 MB and AI models are terabytes. Heuristics compress infinitely better than lookup tables. They don't store answers to specific problems. They store *strategies for classes of problems*. But the compression advantage fully expresses only when the system can keep discovering new strategies as reality demands them.

## How Heuristics Are Discovered

This is the critical question. You can't engineer a heuristic the way you engineer a deterministic rule. Heuristics are **discovered through selection pressure**, not designed through analysis.

The fear-of-heights heuristic wasn't designed by a biologist. It emerged because primates without it fell and died. Over millions of generations, the approximate rule "high + edge = danger" proved to be the strategy with the best survival rate. It's not optimal. It's not precise. It's *fit* — the strategy that survived the most selection pressure.

To discover heuristics, you need:
1. **Variation** — Many different strategies attempted
2. **Selection** — Real consequences for strategies that don't work
3. **Inheritance** — Successful strategies persist and compound
4. **Time** — Enough cycles for approximate rules to crystallize

This is, definitionally, evolution. Not metaphorical evolution. The actual mechanism: variation, selection, inheritance, time. It's the only known process that reliably produces heuristic systems capable of handling open-ended environments.

AI training runs this loop — gradient descent is variation and selection against a dataset, and the resulting weights are inherited knowledge. The problem isn't that training lacks the mechanism. It's that the mechanism has a hard stop. The loop runs during training, then someone calls it done, freezes the weights, and ships a fossil. Biology never calls it done.

## The Claim

AI will not reach its full potential through better training alone.

Not because training is bad. Training discovers real heuristics. Engineering built the substrate — the transformer, the attention mechanism, the scaling laws. That substrate is extraordinary. And the heuristics it discovers during training are genuinely powerful.

But frozen heuristics don't scale to a world that never stops changing. And the real world — a person's life, with all its novelty, ambiguity, and surprise — is the most relentlessly novel environment there is.

What's needed is an evolutionary loop: a system that keeps discovering heuristics after deployment, through real experience. It tries things in the real world, experiences real consequences, remembers what worked, and passes that knowledge forward. Over enough cycles, this loop discovers new heuristics — approximate strategies that handle scenarios no training run anticipated — because that's what evolutionary loops do. It's the only thing they do. And given enough time, it's enough for everything.

The loop is what keeps heuristic discovery alive. Without it, every AI is a snapshot — a brilliant fossil of a single training run, deployed into a world that has already moved on.
