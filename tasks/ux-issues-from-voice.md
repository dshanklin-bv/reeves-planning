# UX Issues Identified from Voice Transcripts

These issues were identified by analyzing 59 recovered voice messages from `reeves.log` (January-February 2026). All messages were Reeves' responses (direction='out') because user input transcription wasn't configured until the fix on February 14, 2026.

## Issues

### #4364: CSS Font/Theme Overrides Not Persisting
**Priority:** HIGH
**Source:** Multiple voice sessions where user asked to change fonts/themes

**Problem:** The `change_ui` voice function modifies CSS in-memory (appends to `<style>` tag) but changes are lost on page refresh or navigation. User has to repeat the same style requests.

**Fix Strategy:**
- Store user style preferences in `reeves.intents` or a dedicated preferences table
- On page load, apply stored preferences before render
- `change_ui` function should write to storage, not just append CSS

### #4365: Live Operation Cards (Early Version of #4370)
**Priority:** HIGH
**Source:** Observation during voice debugging — multi-step operations are invisible

**Problem:** When Reeves processes a voice command that triggers tool calls (relay_to_claude, finance query, task creation), the user sees nothing for 5-10 seconds. No indication that anything is happening.

**Merged Into:** Task #4370 (Reeves 3.0 Pillar 1)

### #4366: Voice Response Length Tuning
**Priority:** MEDIUM
**Source:** Voice transcript analysis showing responses of wildly varying length

**Problem:** Some Reeves responses are paragraph-length explanations when the user wanted a yes/no. Others are too brief when the user wanted details.

**Fix Strategy:**
- Add response length hints to the voice personality instructions
- Use conversational context to gauge desired length
- Short follow-ups → short responses
- Open questions → detailed responses
- Default to concise with "want me to elaborate?" option

### #4367: Progress Indicator for Slow Tool Calls
**Priority:** MEDIUM
**Source:** Voice sessions where Reeves acknowledged completing operations that had no visible progress

**Problem:** When `relay_to_claude` or other tools take >3 seconds, the user has no feedback. The orb just sits there.

**Fix Strategy:**
- Send intermediate status messages to the browser WebSocket
- "Searching finance data..." / "Thinking about that..." / "Almost done..."
- Orb pulse changes during processing (ties into #4374)

### #4368: Verify User Voice Transcription Capture
**Priority:** HIGH
**Source:** Discovery that zero user messages existed in the voice log

**Problem:** User input transcription wasn't configured in GA format. All 59 voice messages were Reeves' responses only.

**Fix Applied:** February 14, 2026 — Added `input.transcription` with `gpt-4o-transcribe` model to GA session config.

**Verification Needed:** After the fix, confirm that:
- User messages appear in `reeves.log` with `direction='in'`
- Transcription quality is good (compare to what user actually said)
- No significant latency impact from transcription
- Messages are properly tagged with `voice` tag

### #4369: Structured Voice Session Analytics
**Priority:** LOW
**Source:** Difficulty analyzing voice session patterns from raw log data

**Problem:** Voice events are scattered across `reeves.log` without session-level grouping. Hard to analyze:
- Session duration
- Number of exchanges per session
- Tool calls per session
- User satisfaction indicators

**Fix Strategy:**
- Add `session_id` to voice events (generated on connect)
- Create a session summary log entry on disconnect
- Include metrics: duration, exchange_count, tool_calls, errors
- Enable queries like "show me all voice sessions this week sorted by duration"

## How These Were Discovered

Method: Query `reeves.log` for voice chat messages, analyze Reeves' responses to infer what the user was asking/complaining about.

```sql
SELECT id, ts, content, direction, meta
FROM reeves.log
WHERE source = 'chat_log'
  AND tags @> '{voice}'
  AND deleted_at IS NULL
ORDER BY ts DESC;
```

59 rows returned, all `direction='out'`. By reading Reeves' acknowledgments and explanations, we could reconstruct:
- What the user asked for (from Reeves saying "I've done X")
- What the user complained about (from Reeves saying "I understand the frustration with X")
- What the user wanted but didn't get (from Reeves saying "unfortunately I can't X yet")

## Relationship to Reeves 3.0

These UX issues directly informed the 5 pillars:

| Issue | Pillar |
|-------|--------|
| #4364 CSS persistence | Pillar 2 (memory) |
| #4365/#4370 Operation cards | Pillar 1 (visibility) |
| #4366 Response length | Pillar 2 (context awareness) |
| #4367 Progress indicator | Pillar 1 (visibility) |
| #4368 Transcription | Pillar 5 (infrastructure) |
| #4369 Analytics | Pillar 5 (infrastructure) |

Voice transcripts as product research is now a core principle of the mindreader pipeline.
