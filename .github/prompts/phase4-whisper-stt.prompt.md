# Phase 4: Whisper Speech-to-Text Integration

## Context
Read SKILLS.md (skill: whisper-transcription), DEVELOPMENT_PHASES.md (Phase 4),
and .github/copilot-instructions.md for full context.

## Task
Integrate local Whisper for transcribing audio windows:

1. `AudioFormatConverter` — converts raw PCM 16kHz/16-bit/mono byte arrays to WAV format
   (add 44-byte WAV header)
2. `WhisperTranscriptionService` — sends WAV as multipart/form-data POST to local Whisper
   HTTP endpoint, parses JSON response for transcript text
3. `TranscriptEntry` record — `(String speakerId, Instant startTime, Instant endTime, String text)`
4. Wire into audio pipeline: `AudioBufferAggregator` flush → `AudioFormatConverter` → `WhisperTranscriptionService`

## Configuration
- `whisper.endpoint` — default `http://localhost:8080/inference`
- `whisper.model` — default `base`
- `whisper.language` — default `en`
- Configurable timeout (default 30s)

## Constraints
- Use Spring `WebClient` for HTTP calls to Whisper
- Proper timeout and error handling (Whisper may be slow or unavailable)
- Unit tests with mocked Whisper responses
- Follow copilot-instructions.md strictly