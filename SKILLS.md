# SKILLS.md – Teams Meeting Summarizer

## Skill: scaffold-spring-boot-project
### Description
Create the initial Spring Boot 4.x Maven project with all required dependencies, configuration files,
package structure, and placeholder classes. The project must compile and start with an empty context.

### Dependencies Required
- `spring-boot-starter-web`
- `spring-boot-starter-websocket`
- `spring-boot-starter-actuator`
- `spring-boot-starter-validation`
- `azure-communication-callautomation:1.6.1`
- `azure-communication-common:1.3.8`
- `azure-identity:1.14.2`
- `spring-cloud-azure-dependencies:7.1.0` (BOM)
- `lombok` (optional, compile-only)
- `spring-boot-starter-test`
- `spring-boot-devtools` (dev only)

### Acceptance Criteria
- Project compiles with `mvn clean compile`
- Application starts with `mvn spring-boot:run`
- Actuator health endpoint responds at `/actuator/health`
- Package structure matches copilot-instructions.md
- Java 21 is set as source/target
- `application.yml` has placeholder config sections for ACS, Whisper, and Ollama

---

## Skill: acs-meeting-join
### Description
Implement the ACS Call Automation integration that joins a Microsoft Teams meeting and initiates
audio streaming to a WebSocket endpoint.

### Components
- `AcsConfig` — `CallAutomationClient` bean from connection string
- `MeetingJoinService` — joins a Teams meeting using `TeamsMeetingLinkLocator`, configures
  `MediaStreamingOptions` for PCM 16kHz mono unmixed audio over WebSocket
- `MeetingController` — REST endpoints: `POST /api/meetings/join`, `POST /api/meetings/{id}/leave`
- `AcsCallbackController` — `POST /api/callbacks` to handle ACS lifecycle events
  (`CallConnected`, `CallDisconnected`, `ParticipantsUpdated`)

### Acceptance Criteria
- Can join a real Teams meeting by posting a meeting link
- ACS callback events are received and logged
- Call connection ID is returned and can be used to leave the meeting
- Errors return RFC 7807 ProblemDetail responses

---

## Skill: audio-websocket-handler
### Description
Implement the WebSocket server that receives real-time audio data streamed from ACS during a
Teams meeting. Parse the JSON metadata and binary PCM audio frames.

### Components
- `WebSocketConfig` — registers handler at `/ws/audio-stream`
- `AudioStreamWebSocketHandler` — handles `AudioMetadata`, `AudioData`, `StoppedMediaStreaming` messages
- `AudioChunkProcessor` — receives decoded PCM byte arrays with speaker ID and timestamp
- `AudioBufferAggregator` — accumulates 20ms PCM frames into 30-second windows per speaker

### Audio Format
- PCM 16kHz, 16-bit, mono
- 50 frames/second, 640 bytes per frame
- 30-second window = ~960KB per speaker

### Acceptance Criteria
- WebSocket endpoint accepts connections from ACS
- JSON metadata messages are parsed (kind: AudioMetadata, AudioData, StoppedMediaStreaming)
- Base64-encoded audio data is decoded to raw PCM bytes
- Silent frames are skipped
- Audio is aggregated per-speaker in 30-second windows
- When a window is full, it is flushed for transcription

---

## Skill: whisper-transcription
### Description
Integrate with a locally running Whisper speech-to-text service to transcribe PCM audio windows
into text. Support both whisper.cpp HTTP server and faster-whisper.

### Components
- `WhisperTranscriptionService` — sends PCM audio (as WAV) to local Whisper endpoint, returns transcript text
- `AudioFormatConverter` — converts raw PCM 16kHz 16-bit mono to WAV format for Whisper input
- `TranscriptEntry` record — holds speaker ID, timestamp range, and transcribed text

### Configuration
```yaml
whisper:
  endpoint: "http://localhost:8080/inference"
  model: "base"
  language: "en"
  response-format: "json"
```

### Acceptance Criteria
- PCM audio byte arrays are converted to valid WAV format
- WAV is sent to local Whisper via HTTP multipart POST
- Transcript text is returned and associated with speaker ID and timestamp
- Timeout and error handling for Whisper being unavailable
- Unit tests with mock Whisper responses

---

## Skill: gdpr-health-filter
### Description
Implement a two-pass GDPR Article 9 health data filter for spoken language transcripts.
First pass uses regex pattern matching. Second pass uses Ollama LLM classification for
borderline cases.

### Components
- `GdprFilterService` — orchestrates the two-pass filtering pipeline
- `RegexHealthFilter` — pattern-based detection of medical terms, conditions, medications,
  health phrases common in spoken language, ICD-10 codes
- `LlmHealthClassifier` — sends borderline text to Ollama to classify if it contains health data
- `FilterResult` record — holds original text, redacted text, whether it was filtered,
  matched terms, and filter reason
- `GdprAuditLogger` — logs filter decisions (what was filtered, why, when) without logging
  the redacted content itself

### Spoken Language Patterns (Non-Exhaustive)
- Direct medical terms: diagnosis, medication, surgery, hospital, therapy, etc.
- Spoken health phrases: "feeling unwell", "called in sick", "recovery took", "on medication",
  "mental health", "burnout", "stress leave", etc.
- Context-aware: when health data is detected, also flag the next 1-2 transcript entries
  (likely follow-up discussion on the same topic)

### Acceptance Criteria
- Regex filter catches obvious health-related terms in transcribed speech
- LLM second pass classifies ambiguous text (configurable, can be disabled)
- Context-aware filtering flags surrounding entries when health data is found
- Audit trail is written for every filter decision
- Redacted content is NEVER logged or sent to the LLM
- Comprehensive unit tests with both positive (should redact) and negative (should pass) cases
- False-positive rate is acceptable (prefer over-filtering to under-filtering for GDPR)

---

## Skill: ollama-summarization
### Description
Integrate with a locally running Ollama instance to produce rolling meeting summaries from
filtered transcript text.

### Components
- `OllamaSummaryService` — sends filtered transcript to Ollama `/api/generate` endpoint
  with a structured summarization prompt
- `MeetingSummaryService` — orchestrates the pipeline: filter → summarize → store
- `MeetingSummary` record — holds summary text, topics, action items, decisions,
  open questions, redacted message count
- `RollingConversationBuffer` — maintains a sliding window of filtered transcript entries
  for periodic re-summarization

### Prompt Requirements
The summarization prompt must instruct the LLM to:
- Identify key decisions, action items (with owners), and open questions
- Group content by topic
- Note disagreements or unresolved discussions
- Use speaker names for attribution
- NEVER include health, medical, or personal wellbeing information
- Skip redacted sections entirely

### Configuration
```yaml
ollama:
  base-url: "http://localhost:11434"
  model: "llama3"
  summary-interval-seconds: 300
```

### Acceptance Criteria
- Ollama REST API is called with properly structured prompt
- Summary is parsed into structured sections (topics, decisions, action items, questions)
- Rolling buffer maintains last N minutes of filtered transcript
- Summary is regenerated at configurable intervals
- Timeout and error handling for Ollama being unavailable
- Unit tests with mock Ollama responses

---

## Skill: angular-frontend
### Description
Build the Angular 18+ frontend that allows users to input a Teams meeting link, join/leave
meetings, and view live rolling summaries with GDPR filter statistics.

### Components
- Meeting join page: input for Teams meeting link, join/leave buttons
- Live summary dashboard: displays structured summary (topics, decisions, action items)
- GDPR filter indicator: shows count of redacted messages
- Real-time updates via Server-Sent Events (SSE) from Spring Boot

### Acceptance Criteria
- User can paste a Teams meeting link and trigger join
- Live summary updates appear without page refresh
- GDPR redaction count is visible
- Leave meeting button disconnects cleanly
- Responsive layout

---

## Skill: sse-live-updates
### Description
Implement Server-Sent Events (SSE) endpoint in Spring Boot to push live summary updates
and transcription status to the Angular frontend.

### Components
- `SummaryStreamController` — SSE endpoint at `GET /api/stream/summary/{connectionId}`
- `SummaryEventPublisher` — publishes summary update events when new summaries are generated
- Event types: `summary-update`, `transcript-entry`, `gdpr-redaction`, `meeting-status`

### Acceptance Criteria
- SSE endpoint streams events to connected Angular clients
- New summaries are pushed automatically when generated
- Meeting status changes (joined, participant changes, left) are streamed
- Connection handles client disconnect gracefully