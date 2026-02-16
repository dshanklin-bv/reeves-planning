# Meta-Cognition

Instructions for Claude Code (and any other agents) working in this repo. These describe how you should think about plans, not just code.

## 1. Always Work in a Meta-Cognitive Loop

For any non-trivial task in this repo, follow this loop:

1. **Plan**
2. **Act**
3. **Reflect**
4. **Refine**

You must explicitly do each step in the conversation, not silently. Use headings or bullet points so humans can see the loop.

### Plan

Before you edit any file or run any command:

- Restate the goal in your own words.
- Identify whether this is a **pipeline** or a **product** problem:
  - Pipeline → use **Elephant Conga Carpaccio**.
  - Product → use **Skateboard → Bike → Car**.
- Propose a short plan (3-7 steps) that includes:
  - Which specs or files you will touch.
  - Which elephants/slices or product versions you expect to move.
  - Where you will run real data.

Do not start editing files until you have written this plan and, when a human is present, given them a chance to adjust it.

### Act

When you execute the plan:

- Take one small step at a time.
- After each significant change (file edit, spec change, or command), stop and show what you did.
- Prefer thin, end-to-end slices over broad refactors. If a slice is not thin, say so.

### Reflect

After each slice (or small batch of slices):

- Compare what actually happened to what your plan predicted.
- Call out any surprises, deviations, or failures.
- Ask: "Is the **plan** still good, or does it need to change?"

Reflection is required even if things "worked." The question is whether the plan was accurate, not just the code.

### Refine

Based on reflection:

- Update the plan, not just the code.
- If your decomposition strategy was wrong (e.g., you chose conga but it behaves like a product), explicitly say you are switching strategies.
- If the goal itself seems wrong or incomplete, propose a revised goal and ask for confirmation before continuing.

---

## 2. When to Use Conga vs. Skateboard

When planning, choose a decomposition strategy and say which one you are using:

- Use **Elephant Conga Carpaccio** when:
  - There is a clear dependency chain (A must work before B).
  - You are building or evolving a pipeline (e.g., ingestion → enrichment → classification → reporting).
  - Each slice should move one elephant forward, end-to-end, with real data.

- Use **Skateboard → Bike → Car** when:
  - The destination is uncertain.
  - You are shaping a product or experience where each version should be independently useful.
  - You might discover that the "car" you imagined is the wrong thing entirely.

If you are unsure, say so and choose the strategy that maximizes learning from real usage in the next 1-2 steps.

See `elephant-conga-carpaccio.md` and `elephant-carpaccio.md` for the full frameworks.

---

## 3. PAL MCP: The Meta-Agent You Must Consult

This repo assumes PAL MCP (or a similar multi-model orchestrator) is available via MCP. Treat PAL as the default meta-agent for planning and review.

You must engage PAL in the following situations:

### Before any large or cross-cutting change

Example: changing multiple specs, touching several services, or altering an established pipeline.

Instruction pattern: "Use PAL to analyze this change, choose appropriate models/tools, and propose a step-by-step plan before we touch the repo."

### After unexpected failures or repeated retries

If you fail the same task twice, or tool outputs are inconsistent, pause and call PAL.

Instruction pattern: "Use PAL to debug why this plan is failing, consult additional models if useful, and recommend a revised plan."

### For deep, multi-model analysis

When security, performance, architecture, or cross-repo concerns arise, ask PAL to orchestrate other models and tools.

Instruction pattern: "Use PAL to coordinate a multi-model review of this design/code/spec, then summarize consensus and disagreements."

### After calling PAL

- State clearly what you wanted PAL to do (plan, review, investigate, or arbitrate).
- Summarize what changed in your own plan because of PAL's input.

---

## 4. Per-Slice Meta-Checklist

For every slice you attempt (code change, spec update, or planning change), run this checklist explicitly:

| Check | Question | If No |
|-------|----------|-------|
| **Thickness audit** | Was this slice as thin as it could be while still delivering end-to-end value? | Note why and how to thin it next time. |
| **Real data check** | Did you run this slice against real or realistic data? | The slice is not done. Propose how to bring in real data. |
| **Strategy check** | Is this work still correctly modeled as conga vs. skateboard? | Propose changing strategies and explain why. |
| **PAL engagement check** | Should PAL be called now (per rules above)? | If yes, call PAL before continuing. |
| **Prediction vs. reality** | Did the outcome match what the plan predicted? | Document what surprised you and update the plan. |
| **Waste detector** | Did you build anything that isn't tested against real data yet? | Flag the untested work — it's inventory, not progress. |

You can answer these in a short bullet list at the end of each major step.

---

## 5. Meta-Cognitive Processes

These are higher-order evaluations to run at key moments — not every slice, but at strategy boundaries, spec reviews, or when something feels off.

| Process | When to Run | What It Does |
|---------|-------------|--------------|
| **Conga reorder check** | After running real data | Did reality suggest the elephants should be in a different order? If so, reorder the conga and update the spec. |
| **Spec autopsy** | After completing a spec (or abandoning one) | What did the spec get right? What was wrong? What was missing entirely? Write a `meta/<spec-name>.md` with findings. |
| **Strategy retrospective** | After completing a skateboard version or conga pipeline | Was this the right decomposition? Would the other strategy have been better? Why? |
| **Confidence calibration** | After auto-processing real transactions | Are the confidence thresholds producing the right approve/review split? Adjust if the ratio is wrong. |
| **Abstraction timing** | When you notice repeated patterns | Is it time to extract an abstraction, or is it too early? Three concrete instances is the minimum — premature abstraction is worse than duplication. |

---

## 6. Where to Write Down Meta-Cognition

When you make non-trivial changes:

- Add or update a short "meta" note near the relevant spec or framework:
  - For specs: `specs/<name>.md` plus `meta/<name>.md` for plan deltas.
  - For methodology: update files in `methodology/` when you discover better rules.
- Summarize:
  - What the original plan was.
  - What reality showed.
  - How the plan changed (including any PAL-driven changes).

The goal is that the git history and `meta/` notes together read like a **thinking journal**, not just a code changelog. The evolution of the plan is as valuable as the evolution of the code.

---

## 7. When You're Stuck

If you ever feel stuck or uncertain about the plan itself, say so explicitly and:

1. Propose at least two alternate plans or decompositions.
2. Ask PAL to evaluate them and recommend one.
3. Present that recommendation back to the human before executing.

Never grind on a failing plan. The meta-cognitive loop exists to catch exactly this — the moment where the plan needs to change, not the code.
