# Trust and Autonomy: How an AI Earns the Right to Act

## The Problem

Every AI system faces the same question: how much should it do on its own?

The industry's answer is binary. Either the AI is a tool (does nothing without instruction) or it's an agent (does everything autonomously). Tools are safe but useless without a human driving. Agents are powerful but terrifying because they act without judgment about whether they *should*.

Both are wrong. The correct model is the one biology uses: **earned autonomy through demonstrated competence.**

A child doesn't start life with the keys to the car. They earn trust through a progression: supervised play → unsupervised play → walking to school alone → driving with a parent → driving alone. Each stage is earned by demonstrating competence at the previous stage. Trust is never granted globally — a kid trusted to walk to school isn't trusted to cook dinner. And trust can be revoked instantly when something goes wrong.

This is how `can_i` works. It's the function that governs every autonomous action in Reeves.

## The Function

```
can_i(action, person, domain, confidence, context) → silent | inform | propose | defer
```

Before any job acts on the world — sends a message, archives a thread, creates a task, spends money — it asks this one question. The answer determines the autonomy level:

- **Silent**: Act, log, move on. The pilot never sees it unless they check the log.
- **Inform**: Act, then tell the pilot what you did.
- **Propose**: Prepare the action, present it ready to fire, wait for one-click approval.
- **Defer**: Cannot act. Present the situation and ask the pilot to decide.

## Trust Is Granular, Not Global

This is where most permission systems fail. They treat trust as a single dial: "AI autonomy: low/medium/high." That's useless. A person might fully trust the system to archive spam (earned through weeks of zero corrections) while wanting to personally approve every message to their mother (because the stakes are different and the relationship is precious).

`can_i` maintains trust at the intersection of three dimensions:

- **Action type**: What is the system doing? Archiving is low-stakes. Sending a message as the user is high-stakes. Spending money is highest-stakes.
- **Person**: Who does this involve? Inner circle relationships have different trust profiles than acquaintances. Unknown contacts always start at Defer.
- **Domain**: Which app is acting? Classifying messages is a different trust context than modifying the calendar.

Each unique combination of (action, person, domain) has its own trust level that evolves independently. The trust surface is high-dimensional, not a single slider.

## Trust Calibration Is Selection Pressure

Here's where this connects to the evolutionary philosophy.

Selection pressure (see `selection-pressure.md`) is what makes information dense and useful. The harshness, duration, and reality-groundedness of the pressure determines quality. DNA is 750 MB because the pressure was death over 3.8 billion years. AI models are terabytes because the pressure was a loss function over months.

Trust calibration applies selection pressure to autonomous actions:

```
Unknown → Defer           (no data, always ask)
    ↓ human approves repeatedly
Calibrating → Propose     (some data, show your work)
    ↓ low override rate over time
Trusted → Inform          (good track record, act and tell)
    ↓ continued zero corrections
Autonomous → Silent       (proven competence, just log)
    ↑ human corrects → drops back
```

Every approval is a **positive selection signal**: "this type of action, for this person, in this domain — the system got it right." Every correction is a **negative selection signal**: "wrong. Recalibrate."

This is Level 3 selection pressure from the selection pressure framework — confidence calibration. The system tracks prediction accuracy over time. High accuracy earns more autonomy. Low accuracy triggers more human review. This IS natural selection applied to individual actions.

The key insight: **corrections are more informative than approvals.** An approval says "you were right this time." A correction says "you were wrong about THIS type of action for THIS person" — it encodes a specific failure mode into the trust surface. Over time, the trust model develops the same density that biological heuristics develop: not by memorizing every case, but by encoding approximate strategies that generalize.

## Heuristic, Not Deterministic

`can_i` is not a permission table. It's a heuristic function (see `heuristic-not-deterministic.md`).

A permission table says: "action X requires approval level Y." It handles scenarios the programmer anticipated. It scales by adding more rows.

`can_i` says: "given everything I know about this action, this person, this domain, and this user's correction history — what's my best guess at the right autonomy level?" It handles scenarios nobody anticipated. It scales by accumulating more signal.

A new contact the system has never seen before? No row in a permission table. But `can_i` has a heuristic: unknown contacts start at Defer, because the user's correction pattern on past unknown contacts showed they want to vet new people. That heuristic was never programmed. It was discovered from the trust surface shaped by lived experience.

This is the difference between engineering safety (enumerate cases, add rules) and evolutionary safety (let the system discover what's safe through feedback). Both are needed — hard boundaries are engineering, trust calibration is evolutionary. But the evolutionary component is what makes the system *adaptive* rather than merely *configured*.

## Hard Boundaries: The Immune System

Biology doesn't rely solely on heuristics. The immune system has hardcoded responses — inflammation, fever, pain withdrawal reflexes. These aren't learned. They're inherited. They exist because the cost of getting them wrong is death, and the probability space is well-understood enough to encode directly.

`can_i` has the same structure:

| Action | Maximum autonomy | Why |
|--------|-----------------|-----|
| Send as user | Propose | Impersonation risk. Always confirm. |
| Spend money | Propose | Financial loss is irreversible. |
| First contact | Propose | Social risk with unknown person. |
| Delete data | Inform | Destructive. At minimum, tell the pilot. |
| Share personal info | Propose | Privacy is non-negotiable. |

These boundaries are absolute ceilings. No amount of successful calibration overrides them. The system could have a perfect track record of 10,000 correct message sends — it still shows the draft before sending as the user.

This maps directly to biological safety: your pain withdrawal reflex doesn't care that you've successfully touched hot things before. Some responses are too important to be learned. They're inherited.

## Jobs Are the Organisms

In the evolutionary loop, organisms interact with the environment, and selection pressure shapes which strategies survive. In Reeves:

- **Jobs** are the organisms. Each job is a unit of work that the system creates, executes, and completes.
- **`can_i`** is the selection pressure. It determines which jobs execute freely, which jobs need approval, and which jobs are blocked until the human decides.
- **Trust calibration** is the inheritance mechanism. What the system learns from corrections and approvals shapes how future jobs are evaluated.

A job that gets corrected is like an organism that fails to reproduce — its strategy doesn't propagate. A job that succeeds reinforces the trust level that allowed it to run, making future similar jobs more likely to execute at the same or higher autonomy level.

Over time, the system evolves toward fitness for one specific person's trust preferences. Not by being programmed, but by living under the selection pressure of that person's feedback.

## The Attention Queue Is Natural Selection in Real Time

The cockpit's attention queue — the 0-5 items the pilot sees — is where selection pressure is applied in real time.

Every item in the queue is a blocked job. The system tried to act, `can_i` said "not without the pilot," and the job is waiting. The pilot's response IS the selection signal:

- **Approve as-is**: Positive signal. This type of action moves toward more autonomy.
- **Modify and approve**: Partial signal. The intent was right, the execution needs calibration.
- **Reject**: Negative signal. This type of action moves toward less autonomy.
- **Ignore (default fires)**: Implicit approval. The pilot trusts the default enough to not intervene.

The queue is compact (0-5 items) because the system's job is to handle everything it can autonomously, presenting only what it genuinely can't decide. A queue that's always full means the system hasn't earned enough trust. A queue that's always empty means the system is either very trusted or not doing anything interesting.

The healthy state: the queue has 1-3 items, all genuinely requiring human judgment, with good context and reasonable defaults. The system is flying the plane. The pilot is making strategic decisions.

## Why This Matters

The difference between a tool and an autonomous system isn't capability. It's trust.

A system that can do anything but doesn't know *when* to do it is dangerous. A system that knows when to act and when to ask — and gets better at that judgment over time — is an autopilot.

`can_i` is the bridge between capability and autonomy. It's what makes Reeves work for a control-conscious user (everything starts at Propose, slow path to Inform) and a trust-forward user (quick calibration, most things reach Silent within weeks). The same system, the same code, shaped by different selection pressure into different trust profiles.

That's not configuration. That's evolution.
