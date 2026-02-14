# OpenAI Realtime API — GA Format Learnings

## Overview

Reeves' voice layer uses the OpenAI Realtime API to enable voice conversations with the orb. The API transitioned from beta to GA (Generally Available) format, requiring significant changes.

## Key Differences: Beta vs GA

### Session Configuration

**Beta format (old):**
```json
{
    "type": "session.update",
    "session": {
        "modalities": ["text", "audio"],
        "voice": "ash",
        "turn_detection": {
            "type": "server_vad"
        },
        "input_audio_transcription": {
            "model": "whisper-1"
        },
        "tools": [...]
    }
}
```

**GA format (current):**
```json
{
    "type": "session.update",
    "session": {
        "modalities": ["text", "audio"],
        "voice": "verse",
        "output": {
            "format": {"type": "audio/pcm", "rate": 24000}
        },
        "input": {
            "format": {"type": "audio/pcm", "rate": 24000},
            "transcription": {
                "model": "gpt-4o-transcribe",
                "language": "en"
            },
            "turn_detection": {
                "type": "server_vad",
                "threshold": 0.5,
                "prefix_padding_ms": 300,
                "silence_duration_ms": 800
            }
        },
        "tools": [...]
    }
}
```

### Key Changes

| Feature | Beta | GA |
|---------|------|-----|
| Audio format | Implicit | Explicit `audio/pcm` with rate |
| Transcription | `input_audio_transcription` at root | `input.transcription` nested |
| Transcription model | `whisper-1` | `gpt-4o-transcribe` |
| Turn detection | Root level `turn_detection` | Nested under `input` |
| Voice options | `ash`, `ballad`, etc. | `verse`, `ash`, `ballad`, etc. |
| Headers | Required `OpenAI-Beta: realtime=v1` | No beta header needed |

## Transcription Configuration

This was the most important fix. Without transcription configured, Reeves captured zero user voice messages — only the assistant's responses.

```python
"input": {
    "format": {"type": "audio/pcm", "rate": 24000},
    "transcription": {
        "model": "gpt-4o-transcribe",
        "language": "en",
    },
    "turn_detection": {
        "type": "server_vad",
        "threshold": 0.5,
        "prefix_padding_ms": 300,
        "silence_duration_ms": 800,
    },
},
```

**Transcription events received:**
- `conversation.item.input_audio_transcription.completed` — User's speech transcribed
- Content stored in `reeves.log` with `direction='in'`, `channel='voice'`

## VAD (Voice Activity Detection) Settings

Using `semantic_vad` for GA:

| Setting | Value | Purpose |
|---------|-------|---------|
| `type` | `server_vad` | Server-side voice detection |
| `threshold` | `0.5` | Speech confidence threshold |
| `prefix_padding_ms` | `300` | Keep 300ms audio before speech start |
| `silence_duration_ms` | `800` | Wait 800ms of silence before end of turn |

## Function Calling

Five tools available to the voice model:

| Function | Purpose |
|----------|---------|
| `relay_to_claude` | Send complex queries to Claude for processing |
| `change_ui` | Modify UI elements (theme, layout, style) |
| `navigate_page` | Navigate to a different page in Reeves |
| `toggle_panel` | Show/hide panels (chat, tasks, finance) |
| `switch_tab` | Switch between tabs in a panel |

Function call flow:
1. OpenAI sends `response.function_call_arguments.done`
2. Voice bridge dispatches to appropriate handler
3. Result sent back to OpenAI as `conversation.item.create` (tool result)
4. For `relay_to_claude`: message queued in `claude_queue`, processed by worker

## Audio Relay

Browser → OpenAI:
- `{"type": "audio", "audio": "<base64 PCM16>"}` → `input_audio_buffer.append`
- `{"type": "commit"}` → `input_audio_buffer.commit` (with buffer-too-small guard)
- `{"type": "text", "text": "hello"}` → `conversation.item.create`
- `{"type": "cancel"}` → `response.cancel`

OpenAI → Browser:
- `response.audio.delta` → `{"type": "audio", "audio": "..."}`
- `response.audio.done` → `{"type": "audio_done"}`
- `input_audio_buffer.speech_started` → `{"type": "speech_started"}`
- `input_audio_buffer.speech_stopped` → `{"type": "speech_stopped"}`

## Structured Logging

Added in this session for debugging and analytics:

```python
logger.info(f"TRANSCRIPT [{role}]: {content[:200]}")
logger.info(f"FUNCTION_CALL: {name}({args_summary}) call_id={call_id[:12]}")
logger.info(f"RESPONSE_DONE: output_items={len(items)} usage={usage}")
logger.info(f"SESSION_UPDATED: voice=verse, transcription=gpt-4o-transcribe")
```

## Debugging Notes

### Problem: Zero User Transcripts
- **Symptom:** 59 voice messages in log, all `direction='out'` (assistant), zero `direction='in'` (user)
- **Root cause:** `input_audio_transcription` was never configured in GA format (beta format returned "Unknown parameter" error)
- **Fix:** Added `transcription` config nested inside `input` with `gpt-4o-transcribe` model

### Problem: Buffer Too Small Guard
- **Symptom:** Single audio chunks followed by immediate commit would trigger excessive responses
- **Fix:** `commit_counter >= 3` guard — only send `input_audio_buffer.commit` if at least 3 audio chunks have been received

### Problem: Stale Log Messages
- **Symptom:** Logs said `voice=ash` when voice was changed to `verse`
- **Fix:** Updated log message to `voice=verse, transcription=gpt-4o-transcribe`

## Model: gpt-4o-realtime-preview-2025-06-03

Current model in use. WebSocket URL:
```
wss://api.openai.com/v1/realtime?model=gpt-4o-realtime-preview-2025-06-03
```

Connection established with:
```python
async with websockets.connect(
    f"{REALTIME_URL}?model={REALTIME_MODEL}",
    additional_headers={
        "Authorization": f"Bearer {OPENAI_API_KEY}",
    },
) as openai_ws:
```

No beta headers needed for GA.
