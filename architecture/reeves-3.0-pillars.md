# Reeves 3.0 — Five Engineering Pillars

Project #4585 in Reeves. 11 tasks across 5 pillars transforming Reeves from a powerful-but-invisible backend into the first true personal AI operating system.

## The Three Gaps

Analysis of the current system revealed three fundamental gaps:

1. **Continuity Gap** — Voice sessions are one-shot. No cross-session memory. Each conversation starts cold. The orb doesn't remember what you said yesterday.

2. **Visibility Gap** — The AI does powerful things (brain recommendations, sleep consolidation, graph traversal) but none of it is visible. The user has no idea what Reeves is thinking or doing.

3. **Agency Gap** — Reeves is reactive. It waits for commands. A true AI OS should proactively surface what you need before you ask.

The five pillars close these gaps.

## Pillar 1: Visible Intelligence

**Goal:** Make Reeves' thinking visible to the user in real-time.

### Task #4370: Build Live Operation Cards
- Floating cards near the orb showing concurrent operations
- Multi-track state: "Searching finance..." / "Checking calendar..." / "Querying brain..."
- Cards appear when tools fire, show progress, collapse when done
- Visual proof that Reeves is working, not just silent

### Task #4374: Ambient Orb State
- Orb changes color/pulse based on system state
- Idle = gentle breathe, Processing = focused pulse, Listening = bright steady
- PEFM urgency level reflected in orb warmth
- No interaction needed — ambient information at a glance

### Task #4380: ARIA Accessibility
- Live regions for screen readers
- Operation cards announced as they appear
- Voice transcript visible alongside audio
- Keyboard navigation for all orb interactions

## Pillar 2: Cross-Session Memory

**Goal:** Voice conversations build on previous ones. Reeves remembers.

### Task #4373: Cross-Session Voice Memory
- On voice session start, load last 5 session summaries
- Inject into OpenAI Realtime system prompt as "recent context"
- On voice session end, generate and store session summary
- "Yesterday you mentioned the LLC dissolution" — naturally continuing

### Task #4372: Inject Intents/Facts/PEFM into Voice Prompt
- Query active intents (user's goals, values, constraints)
- Query relevant facts (biographical data)
- Include PEFM emotional state snapshot
- Reeves knows WHO you are, not just WHAT you said

## Pillar 3: Proactive Intelligence

**Goal:** Reeves acts before you ask.

### Task #4375: Proactive Task Suggestions
- Sleep cycle identifies stalled tasks (mentioned but not started)
- Brain recommends actions based on patterns
- Surface suggestions through the orb: "You mentioned X three times this week. Should I break it down?"
- Triage system prioritizes by urgency, frustration, and avoidance duration

### Task #4376: Relationship Maintenance Alerts
- Webb tracks last contact dates
- Configurable cadence per person (desired_cadence_days)
- Alert when relationships are drifting: "You haven't talked to [person] in 47 days"
- Draft reply suggestions from message context

## Pillar 4: Self-Improving Infrastructure

**Goal:** Reeves gets smarter without manual intervention.

### Task #4377: Pattern Extraction Pipeline
- Sleep cycles mine voice transcripts for UX complaints
- Cross-reference with task activity and telemetry
- Auto-generate improvement tasks from patterns
- Close the loop: problem observed → task created → built → verified

### Task #4378: Confidence Calibration
- Track brain recommendation accuracy over time
- Adjust confidence thresholds based on outcomes
- Higher accuracy = more autonomous action
- Lower accuracy = more human review gates

## Pillar 5: Developer Experience

**Goal:** Make it easy to extend and maintain Reeves.

### Task #4379: Structured Telemetry Pipeline
- Every voice event, tool call, and navigation tracked
- Batched writes via async queue (no blocking the user)
- PEFM dimensions computed on write
- Queryable via log_search for debugging and analysis

### Task #4371: Voice Testing Infrastructure
- WebSocket test helpers for voice bridge
- Synthetic audio generation (no microphone needed)
- Mock OpenAI Realtime API responses
- Coverage for all 5 function call types + audio relay + transcription

## Implementation Priority

```
Phase 1 (Week 1-2):
  Pillar 5: Testing + Telemetry (foundation)
  Pillar 2: Cross-Session Memory (highest user impact)

Phase 2 (Week 3-4):
  Pillar 1: Visible Intelligence (the "wow" factor)
  Pillar 3: Proactive suggestions (the "magic" factor)

Phase 3 (Week 5+):
  Pillar 4: Self-improving pipeline (compound returns)
  Accessibility + polish
```

## Success Criteria

A person talks to the orb for 10 minutes. They see live cards tracking 3 concurrent operations. They get a proactive reminder about an overdue task. They ask "what about that tax thing from yesterday?" and get a contextual answer. They say "build that feature" and Claude Code starts with full context.

They never wrote a prompt. They never configured anything. They just lived their life near the system, and the system got smarter.
