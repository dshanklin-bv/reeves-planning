# The Realistic Timeline

*Code today, specs tomorrow, DNA eventually — and why each stage matters.*

---

## The Honest Position

AI cannot self-improve yet. The evolutionary loop is being built, not running at scale. Until it is, humans are the selection pressure. Humans write the specs. Humans evaluate the outcomes. Humans decide what to keep and what to discard.

This means code — deterministic, human-readable, testable code — is necessary for as long as AI depends on human judgment to improve. And humans work on a slower timescale than the vision implies. You don't leap from "AI writes code" to "AI writes DNA" overnight. There's a path, and it has stages.

## The Three Eras

### Era 1: Code (Now — 2025-2028)

**Where we are.** AI writes code under human supervision. Humans prompt, review, test, and deploy. The AI is a powerful tool but a static one — it doesn't improve from use.

**What's happening:**
- AI-assisted development (Copilot, Claude Code, Cursor)
- Human engineers are still the selection pressure
- Code is the artifact: functions, classes, APIs, tests
- People are ramping up on AI through what they understand: writing and reviewing code

**Why this era matters:**
This is where humans learn to trust AI. Not through philosophy papers about evolution. Through the daily experience of "I asked it to write a function and it worked." Trust is built incrementally, through thousands of small successful interactions.

People who dismiss this era ("code generation is trivial") miss the point. The point isn't the code. The point is the relationship. Humans are learning to collaborate with AI. AI is (through RLHF and deployment feedback) getting shaped by human judgment. The loop is slow and manual, but it's running.

**An evolutionary biologist would see:** Artificial selection. Humans manually choosing which AI outputs survive (merge to main) and which die (rejected in review). Exactly how we domesticated wolves into dogs — slow, deliberate, human-guided selection.

### Era 2: Specs (2027-2032)

**Where we're heading.** AI generates not just code but the blueprints — PRDs, architecture docs, test strategies, acceptance criteria. Humans review at the spec level, not the implementation level.

**What's happening:**
- AI writes specs, AI implements specs, humans review outcomes
- The Ralph Loop: same spec fed repeatedly, agent self-corrects
- Multi-agent pipelines: PAL reviews Claude's work, multiple perspectives
- Persistent memory across sessions, but still human-curated
- Selection pressure shifts from "is this code correct?" to "is this spec producing good outcomes?"

**Why this era matters:**
This is where the leverage explodes. A human reviewing a PRD can influence 1,000 lines of code with one paragraph of feedback. The human moves up the abstraction stack. Instead of line-by-line code review, they review *direction*. Instead of writing functions, they describe *intent*.

This is also where non-engineers start participating. A product manager who can't code can write a spec. A domain expert who understands their field can describe what success looks like. The spec is the interface between human judgment and AI capability. Code was too low-level for most people. Specs are accessible.

**An evolutionary biologist would see:** The emergence of a second inheritance channel. Code is genetic inheritance (precise, slow, deterministic). Specs are cultural inheritance (flexible, fast, Lamarckian). Cultural inheritance is what allowed human civilization to outpace genetic evolution by orders of magnitude. Same dynamic here.

### Era 3: DNA (2030+)

**Where evolution leads.** AI systems generate their own specs, test them against reality, keep what works, discard what doesn't. The human role shifts from reviewer to governor — setting constraints, not directing execution.

**What's happening:**
- Systems observe outcomes and generate their own improvement specs
- Sleep cycles extract patterns and encode them as heuristics
- Heuristics are tested against real environments automatically
- Successful heuristics persist and compound
- Failed heuristics decay naturally
- Human governance ensures safety, not direction

**Why this era matters:**
This is where AI becomes genuinely adaptive — where it can handle scenarios nobody anticipated because it discovered heuristics through lived experience. The system writes its own DNA: generative blueprints that build adaptive machinery, tested against reality, compressed by selection pressure.

The human doesn't need to understand the heuristics any more than you need to understand your immune system. You just need to trust that the governance is structural and the outcomes are good.

**An evolutionary biologist would see:** True evolution. Variation (generated specs), selection (outcome testing), inheritance (persistent heuristics), and time (continuous cycles). The loop that produces adaptive intelligence from any starting point, given enough pressure and enough cycles.

## The Transition Points

### Code → Specs
**Trigger:** When AI can reliably implement a spec without line-by-line human guidance.
**Evidence this is starting:** Ralph Loops, multi-agent pipelines, PRD-driven development.
**Human role shift:** From code reviewer to spec author.

### Specs → DNA
**Trigger:** When the system can reliably generate its own specs from observed outcomes.
**Evidence this is starting:** Sleep cycles generating improvement tasks, brain recommendations, proactive intelligence.
**Human role shift:** From spec author to governor.

## Why Each Stage Is Necessary

You can't skip stages. And an evolutionary biologist would tell you exactly why:

**Artificial selection (Era 1) precedes natural selection (Era 3).** We domesticated wolves before we understood genetics. We bred crops before we understood DNA. Deliberate, human-guided selection creates the conditions for autonomous selection. The organisms get good enough — and the trust gets deep enough — for humans to step back.

**Cultural inheritance (Era 2) accelerates genetic evolution.** When humans could pass knowledge between generations through language and writing, evolution didn't stop — it accelerated. The environment became more complex, creating more selection pressure, producing denser adaptations. Specs (cultural inheritance for AI) create the conditions for DNA (autonomous evolution).

**Each era runs concurrently, not sequentially.** Just as cultural and genetic evolution run simultaneously in humans, all three eras will overlap. Era 1 doesn't end when Era 2 begins. Code doesn't disappear when specs emerge. Code becomes what the specs generate. Specs become what the DNA generates. Each layer builds on the layer below.

## The Honest Assessment

**Era 1 is real and working.** Millions of developers use AI to write code today.

**Era 2 is emerging.** Systems like Reeves' maxralphie pipeline, spec-driven development, and self-correcting agent loops are being built by a small number of teams.

**Era 3 is theoretical but grounded.** The individual components (persistent memory, outcome tracking, pattern consolidation, heuristic discovery) exist in prototype. What doesn't exist yet is a system that runs the full loop autonomously at scale.

The gap between Era 2 and Era 3 is where the real work is. It's engineering, not research — but it's hard engineering with open questions about the right selection pressures, the right memory architecture, and the right governance model.

**The retrieval-to-memory loop is a concrete example of the Era 1→2 transition.** Universal search (Era 1: the system observes everything, re-derives answers from scratch) evolves into graph-backed retrieval (Era 2: the system consolidates what it learns, answers compositional questions, gets denser over time). The promotion loop — where retrieval traces become graph edges through sleep-cycle consolidation — is selection pressure applied to knowledge. See `frameworks/search-and-graph.md` for the full treatment.

Biology figured this out through 3.8 billion years of trial and error. We're trying to engineer it deliberately. That's either much harder or much faster, depending on how honest we are about what we don't know yet.

## What an Evolutionary Biologist Would Say

An evolutionary biologist looking at the AI landscape would say:

1. **"You have a powerful replicator. Now give it an environment."** The model is the replicator. It can produce variation. What it needs is an environment with real selection pressure — consequences for its outputs.

2. **"Stop evaluating the organism. Evaluate the loop."** Is there variation? Selection? Inheritance? If yes, intelligence emerges. If no, improve the machinery.

3. **"You're trying to engineer what evolution discovers."** That's not impossible, but be humble about it. Evolution had 3.8 billion years and trillions of parallel experiments. You have funding and a timeline.

4. **"The cultural inheritance channel is your cheat code."** Lamarckian inheritance (acquired traits passed directly) is orders of magnitude faster than genetic inheritance. AI has this naturally — a learned pattern CAN be directly encoded for the next cycle. Biology can't do this. Use that advantage.

5. **"Don't try to build intelligence. Build the conditions for intelligence to emerge."** You can't engineer a brain from first principles. You can engineer an environment where brains evolve. Same with AI: build the loop, set the constraints, provide the pressure, and let it run.

The non-biologist sees the problem as: "How do I make AI smarter?"
The biologist sees the problem as: "How do I make AI's environment more demanding?"

Same question. Profoundly different answer. And the biologist's answer is the one that scales.
