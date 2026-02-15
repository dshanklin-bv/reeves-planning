# Governance: The Three Pillars

The loop says "testing and constraints make speed safe." This is the structure that makes that concrete.

## The Pillars

### Context (what the system has)
What resources and information are available right now.

- Which services are running (from daemon status)
- Which data sources are connected and current
- LLM budget remaining (daily/monthly cost caps)
- Available tools and integrations (MCP registry)
- Current person model quality (graph density, PEFM coverage, embedding freshness)

### Intent (what the system should achieve)
What the system is actively trying to do for the person.

- Per-service goals with executable verifiers (not just text descriptions)
- Active /mindreader insights being acted on
- Scheduled jobs and their expected outcomes
- Deadlines and time-sensitive obligations
- Priority ranking across all active intents

### Guardrails (what the system must never do)
Hard constraints that no agent, job, or automation can violate.

- Never send a message without human approval
- Never delete data (soft deletes only)
- Never exceed $X/day on LLM inference
- Never write to another service's schema
- Never create standalone launchd plists
- Never import the anthropic SDK directly
- All graph mutations require human approval before ingestion

## Approval Lifecycle

```
Intent defined (human or brain)
  → Plan generated (agent)
  → Approval gate (policy-dependent)
  → Execution (daemon job or agent)
  → Verification (executable verifier checks outcome)
  → Feedback (success/failure feeds back to LEARN)
```

Approval policies per intent:
- `none` — fully autonomous (e.g., transaction categorization with >95% confidence)
- `on_failure` — auto-approve, human reviews failures
- `on_plan` — human approves the plan, execution is autonomous
- `on_every_step` — human approves each step (financial transactions, external communications)

## Circuit Breaker

After 3 consecutive failures on an intent → stall the intent and alert. Don't keep retrying. The meta-brain (a governance agent that monitors the brain) evaluates whether the approach is wrong, not just the execution.

## Audit Trail

Every governance decision is logged: who approved, when, what evidence was presented, what the outcome was. This is the same pattern as taskr-ui's governance page — visible, auditable, human-governable.
