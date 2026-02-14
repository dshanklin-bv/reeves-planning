# /mindreader — Know Me Better Than I Know Myself

Mindreader doesn't ask what to do. It already knows.

It reads everything Reeves knows about you — voice, finance, relationships, tasks, emotions, patterns, messages, calendar, habits — and synthesizes a picture of what you actually need right now. Not what you said. What the evidence says.

Then it acts.

## The Stack

```
/mindreader   → Knows what you need (synthesis)
/maxralphie   → Builds what you need (execution)
/ralphie      → Ships what you need (implementation)
```

Mindreader is Phase 0 of everything. It's the answer to "what should I work on?" that nobody asked.

## Arguments

`/mindreader [--focus AREA] [--since DURATION] [--depth shallow|deep|god] [--act]`

- **--focus** — Optional lens: `life`, `work`, `money`, `people`, `health`, `all` (default: all)
- **--since** — How far back to look (default: 30d)
- **--depth** — How deep to go:
  - `shallow` — Quick scan, 2 minutes, top 3 insights
  - `deep` — Full analysis, 5-10 minutes, comprehensive
  - `god` — Everything. Cross-reference all data sources. Find what you're avoiding.
- **--act** — Don't just tell me. Create tasks, write the vision doc, queue the work.

## What It Reads

### Layer 1: What You Said (Explicit Signal)

**Voice transcripts** (reeves.log WHERE source='chat_log' AND tags @> '{voice}'):
- Feature requests you made casually
- Complaints about things not working
- Questions you asked (what you're curious about)
- Things you told Reeves to do but never followed up on
- Tone patterns — when were you excited? frustrated? tired?

**Task backlog** (reeves_task_list):
- Tasks you created but never started (procrastination signal)
- Tasks you started but abandoned (friction signal)
- Tasks you completed quickly (energy signal — what flows?)
- Recurring tasks you keep snoozing (avoidance signal)

**Devlogs** (devlog_search):
- What you've been building (focus areas)
- What broke and how you fixed it (pain points)
- Decisions you made and why (values signal)
- Patterns tagged `improvement` or `idea` (latent ambition)

### Layer 2: What You Did (Behavioral Signal)

**Finance** (finance data):
- Spending patterns — where is money going?
- Unusual transactions — something new happening?
- Recurring charges — subscriptions you forgot about?
- Account balances — are you healthy? stressed?
- Tax obligations — deadlines approaching?

**Calendar** (Outlook):
- What's coming up — deadlines, meetings, obligations?
- How packed is this week vs last?
- Meetings you keep rescheduling (avoidance signal)
- Free time blocks (opportunity signal)

**Messages** (Tosh — Apple Messages):
- Who you're talking to most (relationship priorities)
- Conversations that went unanswered (dropped threads)
- Emotional tone of recent exchanges
- People who reached out that you haven't replied to

**Browser/UI telemetry** (telemetry events):
- Pages you visit most (what you check obsessively)
- Features you never use (what's not working for you)
- Time of day patterns (when are you active?)

### Layer 3: What Reeves Knows (AI Signal)

**PEFM emotional state** (knowledge_vectors):
- High urgency items — what's time-sensitive?
- High frustration items — what's been painful?
- High insight items — what have you learned?
- High novelty items — what's new and interesting?
- Decaying satisfaction — what wins are fading?

**Sleep briefings** (log WHERE tags @> '{sleep-generated}'):
- Patterns extracted from your behavior
- Cross-project themes you don't see
- Anomalies — something changed and you didn't notice

**Brain recommendations** (brain.think):
- What does the AI think you should work on?
- Pending governance approvals blocking progress
- Confidence levels — what is the brain sure about?

**Knowledge graph** (relationship_edges):
- Relationship clusters — who's connected to what?
- Temporal gaps — relationships that went cold
- Professional network — opportunities, obligations
- Family/personal — birthdays, milestones coming up

**Intents & facts** (intents, facts tables):
- Your stated values and preferences
- Biographical data that constrains decisions
- Goals you set for yourself

### Layer 4: What You're Avoiding (Shadow Signal)

This is the god-depth layer. Cross-reference everything to find:

- **Procrastination patterns**: Tasks created >14 days ago, mentioned in voice, never started. "You've mentioned the NC tax dissolution 4 times but haven't taken a single step."

- **Relationship drift**: People in your graph with no recent Tosh messages, no calendar events. "You haven't talked to [person] in 47 days. Last time you talked about [topic]."

- **Financial blind spots**: Recurring charges you've never reviewed. Accounts with low activity. "You're paying $X/month for [service] — last used [date]."

- **Energy misallocation**: Time spent on low-PEFM tasks vs high-PEFM tasks avoided. "You spent 6 hours on CSS tweaks but the W-2 filing (high urgency, high frustration) hasn't moved."

- **Broken promises**: Things you told people you'd do (from Tosh messages, voice transcripts) that don't have corresponding tasks. "You told [person] you'd [action] on [date]. No task exists for this."

- **Health signals**: Calendar gaps (no breaks scheduled), late-night activity patterns, weekend work frequency. Not medical — behavioral.

## The Output

### Shallow (2 minutes)

```
MINDREADER — Quick Read
━━━━━━━━━━━━━━━━━━━━━━

Top 3 things that need your attention:

1. NC TAX DISSOLUTION — 3 weeks stalled. Mentioned 4x in voice.
   No tax clearance needed (you learned this). Next step: file member consent.
   Urgency: HIGH (deadline in 10 days)

2. [PERSON] HASN'T HEARD FROM YOU — 47 days. Last: discussed [topic].
   They reached out Feb 1, you didn't reply.

3. MERCURY ACCOUNT — $4,200 sitting in AI Seed Holdings.
   No transactions in 60 days. Review or reallocate.

Energy read: You're in a BUILD phase (heavy dev activity, lots of orb usage).
             But you're avoiding admin (tax, finance review, relationship maintenance).
             Suggestion: 30 min admin block tomorrow morning before the build energy kicks in.
```

### Deep (5-10 minutes)

Everything in shallow, plus:
- Full task audit with procrastination scoring
- Finance health summary
- Relationship status map (who's warm, who's cold)
- PEFM emotional trajectory chart (what's rising, what's fading)
- Sleep pattern insights
- Calendar optimization suggestions
- 5-7 specific action items with effort estimates

### God (full synthesis)

Everything in deep, plus:
- Cross-domain pattern analysis (finance + relationships + tasks = life themes)
- Behavioral prediction (based on past patterns, what will you procrastinate on next?)
- Values alignment check (are your actions matching your stated intents?)
- Shadow signal report (what you're avoiding and why)
- Strategic recommendations (not just "do X" but "here's why X matters for your life")
- Vision document draft (if --act is set)

## The Act Flag

When `--act` is set, mindreader doesn't just report. It:

1. **Creates tasks** for every actionable insight
2. **Prioritizes** based on urgency, frustration, and avoidance duration
3. **Writes a vision doc** synthesizing all insights into a coherent "state of your life" document
4. **Queues maxralphie** for any engineering/Reeves improvements identified
5. **Drafts messages** for relationship maintenance (stored as artifacts, you approve before sending)
6. **Schedules admin blocks** in your calendar for avoided tasks
7. **Devlogs everything** — the full analysis becomes durable context for future sessions

## Implementation

### Data Gathering (Parallel — 8 queries simultaneously)

```
Agent 1: Voice transcripts (last 30d)        → reeves.log
Agent 2: Task audit (all enabled tasks)       → reeves_task_list + completions
Agent 3: Finance summary                      → finance endpoints
Agent 4: People status                        → Webb + Tosh data
Agent 5: PEFM emotional state                 → knowledge_vectors
Agent 6: Calendar next 14 days                → Outlook
Agent 7: Sleep + Brain insights               → reeves.log + brain.think
Agent 8: Knowledge graph + Intents + Facts    → graph + intents + facts
```

### Synthesis (PAL + Janus)

Feed all gathered data into two synthesizers in parallel:

**PAL (planner)**: "Given this person's data, what are the top priorities? What patterns do you see? What are they avoiding?"

**Janus (second_opinion)**: "Given this person's stated values (intents) and actual behavior (tasks, finance, messages), where are the gaps? What would a wise advisor say?"

Merge both perspectives into a unified read.

### Output Generation

Format the synthesis into the appropriate depth level. If `--act`, create all action items and artifacts.

## Why Mindreader Exists

Every AI assistant waits for you to ask. Mindreader knows you well enough to tell you what you need to hear.

It's not a task manager. It's not a chatbot. It's the part of your brain that sees the big picture — the part that knows you should call your mom, file your taxes, and stop tweaking CSS when there's real work to do — but usually gets drowned out by the noise.

Reeves has the data. Mindreader has the clarity.

The pipeline:
```
/mindreader              → "Here's what's really going on in your life"
/mindreader --act        → "Here's what's going on, and I've already started fixing it"
/maxralphie              → "Now let's build the tools to make this sustainable"
```

## Example

```
/mindreader --depth god --act

MINDREADER — Full Synthesis
━━━━━━━━━━━━━━━━━━━━━━━━━━

STATE OF YOUR LIFE — February 14, 2026

You're in a creative build phase. Voice orb is working, Reeves 3.0
is taking shape, you're energized by the vision. This is good.

But you're using the build energy to avoid three things:

1. THE LLC DISSOLUTION — 3 weeks stalled, 4 voice mentions, 0 actions.
   The legal work isn't hard. You just don't want to deal with it.
   I've created 5 subtasks breaking it down. First one takes 10 minutes.

2. RELATIONSHIP MAINTENANCE — 3 people haven't heard from you in 30+ days.
   One of them reached out. I've drafted reply suggestions (see artifacts).

3. FINANCIAL REVIEW — Mercury has $4,200 idle. Two subscriptions you
   probably forgot about ($47/mo total). Tax filing deadline in 10 days.
   I've created a 30-minute finance review task for tomorrow morning.

PATTERN I NOTICED:
You tend to build tools to solve problems instead of solving the problems
directly. The orb, the cards, maxralphie — these are all tools. The NC
dissolution doesn't need a tool. It needs 10 minutes and a phone call.

WHAT I DID:
- Created 8 tasks across 3 focus areas
- Drafted 3 message replies (pending your review)
- Scheduled 30-min admin block for tomorrow 9am
- Wrote vision doc: ".claude/mindreader-reports/2026-02-14.md"
- Queued Reeves 3.0 engineering work for maxralphie

Your move.
```
