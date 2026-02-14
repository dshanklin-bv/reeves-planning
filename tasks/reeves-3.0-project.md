# Reeves 3.0 Project — All Tasks

Project #4585 in Reeves: "Reeves 3.0 — Best-in-Class AI Operating System"

## Task Summary

| ID | Pillar | Priority | Title | Status |
|----|--------|----------|-------|--------|
| 4370 | 1 | HIGH | Build live operation cards near the orb | Pending |
| 4371 | 5 | HIGH | Voice testing infrastructure | Pending |
| 4372 | 2 | HIGH | Inject intents/facts/PEFM into voice prompt | Pending |
| 4373 | 2 | HIGH | Cross-session voice memory | Pending |
| 4374 | 1 | MEDIUM | Ambient orb state signals | Pending |
| 4375 | 3 | MEDIUM | Proactive task suggestions | Pending |
| 4376 | 3 | MEDIUM | Relationship maintenance alerts | Pending |
| 4377 | 4 | MEDIUM | Pattern extraction pipeline | Pending |
| 4378 | 4 | LOW | Confidence calibration system | Pending |
| 4379 | 5 | HIGH | Structured telemetry pipeline | Pending |
| 4380 | 1 | LOW | ARIA accessibility for orb | Pending |

## Detailed Task Descriptions

### #4370: Build Live Operation Cards Near the Orb
**Pillar:** 1 (Visible Intelligence)
**Priority:** HIGH
**Files:** `orb.js`, `base.html`, `style.css`

Floating cards near the orb that show concurrent operations in real-time. When Reeves calls a tool or starts a multi-step operation, a card appears showing what's happening. Cards show progress and collapse when done.

**Acceptance Criteria:**
- Cards appear when voice triggers tool calls
- Multiple cards can be visible simultaneously
- Cards show operation name + brief status
- Cards auto-dismiss after completion (with 2s fade)
- Cards are positioned near the orb, don't occlude it

### #4371: Voice Testing Infrastructure
**Pillar:** 5 (Developer Experience)
**Priority:** HIGH
**Files:** `tests/test_voice.py`, `conftest.py`

Comprehensive test coverage for the voice bridge. WebSocket mocking, synthetic audio generation, function call dispatch testing. See `research/voice-transcripts.md` for the specific test strategies (8 test groups).

**Acceptance Criteria:**
- WebSocket connect/disconnect tests pass
- Audio relay tests (browser → OpenAI, OpenAI → browser)
- All 5 function call types tested
- Session config validated (tools, voice, transcription)
- Telemetry emission verified
- No dependency on actual microphone or OpenAI API

### #4372: Inject Intents/Facts/PEFM into Voice Prompt
**Pillar:** 2 (Cross-Session Memory)
**Priority:** HIGH
**Files:** `voice.py`

When a voice session starts, query the user's active intents (goals, values, constraints), relevant facts (biographical data), and current PEFM emotional state. Inject into the OpenAI Realtime system prompt so Reeves knows WHO it's talking to.

**Acceptance Criteria:**
- Active intents queried and summarized
- Relevant facts included in voice prompt
- PEFM snapshot (top 3 emotional dimensions) included
- System prompt stays under 2000 tokens
- Falls back gracefully if data sources are empty

### #4373: Cross-Session Voice Memory
**Pillar:** 2 (Cross-Session Memory)
**Priority:** HIGH
**Files:** `voice.py`

On voice session start, load summaries from the last 5 voice sessions. On session end, generate and store a summary. Inject recent session summaries into the Realtime system prompt.

**Acceptance Criteria:**
- Session summaries stored in `reeves.log` (source='voice_summary')
- Last 5 summaries loaded on session start
- Summaries injected into system prompt naturally
- "Yesterday you mentioned X" works in conversation
- Summary generation uses Claude (via relay_to_claude) at session end

### #4374: Ambient Orb State Signals
**Pillar:** 1 (Visible Intelligence)
**Priority:** MEDIUM
**Files:** `orb.js`, new API endpoint

The orb changes color and pulse pattern based on system state:
- Idle: gentle breathe (current)
- Processing: focused pulse
- Listening: bright steady
- Thinking: rhythmic pulse
- Error: brief red flash

Optional: PEFM urgency level reflected in orb warmth (cool blue → warm amber).

### #4375: Proactive Task Suggestions
**Pillar:** 3 (Proactive Intelligence)
**Priority:** MEDIUM
**Files:** `brain.py`, `triage.py`, voice prompt

Sleep cycle identifies stalled tasks. Brain recommends actions. Surface through the orb proactively: "You mentioned X three times this week. Should I break it down?"

### #4376: Relationship Maintenance Alerts
**Pillar:** 3 (Proactive Intelligence)
**Priority:** MEDIUM
**Files:** `webb.py`, notification system

Webb tracks last contact dates. Alert when inner_circle/friend relationships drift past their desired cadence. Draft reply suggestions from message context.

### #4377: Pattern Extraction Pipeline
**Pillar:** 4 (Self-Improving)
**Priority:** MEDIUM
**Files:** `sleep.py`, `brain.py`

Sleep cycles mine voice transcripts for UX complaints. Cross-reference with task activity and telemetry. Auto-generate improvement tasks from patterns.

### #4378: Confidence Calibration System
**Pillar:** 4 (Self-Improving)
**Priority:** LOW
**Files:** `brain.py`

Track brain recommendation accuracy over time. Adjust confidence thresholds based on outcomes. Higher accuracy → more autonomous action.

### #4379: Structured Telemetry Pipeline
**Pillar:** 5 (Developer Experience)
**Priority:** HIGH
**Files:** `telemetry.py`, `voice.py`

Ensure every voice event, tool call, and navigation is tracked. Batched async writes. PEFM dimensions computed on write. Queryable via log_search.

### #4380: ARIA Accessibility for Orb
**Pillar:** 1 (Visible Intelligence)
**Priority:** LOW
**Files:** `base.html`, `orb.js`

ARIA live regions for screen readers. Operation cards announced as they appear. Keyboard navigation for orb interactions.

## Implementation Priority

```
Week 1-2:
  #4379 Structured Telemetry (foundation for everything)
  #4371 Voice Testing (safety net)
  #4373 Cross-Session Memory (highest user impact)
  #4372 Inject Intents/Facts/PEFM (enriches voice)

Week 3-4:
  #4370 Live Operation Cards (visible intelligence)
  #4374 Ambient Orb State (polish)
  #4375 Proactive Suggestions (magic factor)

Week 5+:
  #4376 Relationship Alerts
  #4377 Pattern Extraction
  #4378 Confidence Calibration
  #4380 Accessibility
```
