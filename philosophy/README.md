# Reeves Philosophy

The philosophical foundation for building AI systems that evolve, not just execute.

## What This Is

This section captures a body of ideas about why current AI is powerful but incomplete, what's actually missing, and how to build the machinery for artificial intelligence that gets better by living — not by being redesigned.

These ideas emerged from building Reeves, a personal AI operating system, and noticing that the hardest problems weren't engineering problems. They were problems that biology solved billions of years ago and computer science hasn't addressed yet.

## Core Arguments

### 1. The Krebs Cycle
Next token prediction is the metabolic substrate of AI — essential, but not intelligence itself. Every cell runs the Krebs cycle. No one points at it and says "that's life."

### 2. The 750 MB Argument
Human DNA encodes the most sophisticated intelligence on Earth in 750 megabytes. AI models use terabytes and can't remember yesterday. The difference isn't size. It's that every byte of DNA was forged against reality over 3.8 billion years.

### 3. Priors + Memory
Intelligence = evolutionary priors (DNA / model weights) + lived experience (memory / the loop). Both are required. A genome with no life is a baby. Life with no priors is impossible.

### 4. Heuristic, Not Deterministic
Evolution produces heuristic systems — approximate rules that generalize to scenarios that have never existed. Engineering produces deterministic systems — precise rules that handle known scenarios. The real world is non-deterministic. Only heuristics scale to infinity.

### 5. DNA Is a Spec, Not Code
DNA doesn't say "do X when Y happens." It says "build a system that figures out what to do." 750 MB doesn't encode behavior. It encodes the machinery for generating behavior. The system builds itself from the spec, shaped by whatever environment it lands in.

### 6. Write DNA, Not Code
Stop using AI to write deterministic instructions. Start using AI to write generative specs — blueprints for systems that build themselves through lived experience and selection pressure.

## Repo Structure

```
philosophy/
├── README.md
├── essays/
│   ├── ai-hasnt-lived-yet.md          # The main article
│   ├── the-krebs-cycle.md             # Substrate vs. intelligence
│   ├── 750-megabytes.md               # Information density argument
│   ├── heuristic-not-deterministic.md  # Why evolution beats engineering
│   └── write-dna-not-code.md          # Specs over code
├── frameworks/
│   ├── two-primitives.md              # Priors + Memory = Intelligence
│   ├── evolutionary-loop.md           # OBSERVE → MODEL → INFER → COLLAPSE → GOVERN → LEARN
│   ├── selection-pressure.md          # What makes AI actually improve
│   ├── search-and-graph.md           # Search is sensing, graph is memory
│   └── ensemble-evolution.md          # Randomized critics as artificial selection
│   (Development methodology frameworks live in ../methodology/)
└── implications/
    ├── scaling-is-not-enough.md        # Why bigger models miss the point
    ├── the-evaluation-error.md         # Judging organisms vs. machinery
    ├── democratization-of-agency.md    # What this means for everyone
    └── privacy-and-consent.md          # The surveillance question, answered
```

## Origin

These ideas crystallized during a design conversation on February 14, 2026, while building Reeves 3.0. The conversation started with engineering ("how do we make this a better app?") and ended somewhere much deeper ("how does a computer learn to know a person?"). The answer turned out to be the same answer biology found 3.8 billion years ago.

## How This Repo Practices What It Preaches

These docs went through the process they describe. They were written, then subjected to multi-model critique — Claude, GPT, Perplexity — each acting as selection pressure on the ideas.

Claims that couldn't survive scrutiny were cut or rewritten. The "100-500 people" grandiosity was identified and removed. The heuristic/deterministic framing was sharpened. A missing privacy argument was filled. The genome claims overstated how optimized DNA is and were corrected to acknowledge the noise while preserving the argument.

What remains is denser than what was first written. Not because someone planned it better, but because it survived pressure. The spec — the core ideas — persisted. The expression — the text — was rebuilt around them.

That's the loop. These docs are a running example of the philosophy they describe.
