# Phase 3: Audio Stream WebSocket Handler

## Context
Read SKILLS.md (skill: audio-websocket-handler), DEVELOPMENT_PHASES.md (Phase 3),
and .github/copilot-instructions.md for full context.

## Task
Implement the WebSocket server that receives real-time audio from ACS:

1. `WebSocketConfig` — registers `AudioStreamWebSocketHandler` at `/ws/audio-stream`
2. `AudioStreamWebSocketHandler` — parses incoming messages by `kind`:
   - `AudioMetadata` — log stream format (sample rate, channels, encoding)
   - `AudioData` — decode base64 PCM, extract participantRawId and timestamp, skip silent frames
   - `StoppedMediaStreaming` — log stream end
3. `AudioChunkProcessor` — receives decoded PCM bytes with speaker ID and timestamp
4. `AudioBufferAggregator` — accumulates 20ms frames (640 bytes each at 16kHz/16-bit/mono)
   into 30-second windows per speaker (~960KB), flushes when window is full

## Audio Format Details
- PCM 16kHz, 16-bit, mono
- 50 frames/second × 640 bytes/frame
- 30 seconds = 1500 frames = ~960KB

## Constraints
- Thread-safe buffer aggregation (ConcurrentHashMap)
- Follow copilot-instructions.md strictly