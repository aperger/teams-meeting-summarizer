# Development Phases – Teams Meeting Summarizer

## Phase 1: Project Foundation
**Goal:** Compilable, runnable Spring Boot 4.x skeleton with correct structure.

### Tasks
- [ ] Scaffold Maven project with Spring Boot 4.0.x parent POM
- [ ] Configure Java 21, Jakarta EE 11, Spring Framework 7
- [ ] Set up package structure per copilot-instructions.md
- [ ] Create `application.yml` with placeholder configuration sections
- [ ] Add all Maven dependencies (ACS SDK, Azure Identity, WebSocket, etc.)
- [ ] Configure Spring Actuator health endpoint
- [ ] Create global exception handler with ProblemDetail (RFC 7807)
- [ ] Verify: `mvn clean compile` and `mvn spring-boot:run` succeed

### Skill Reference
→ `scaffold-spring-boot-project`

---

## Phase 2: ACS Integration – Join Teams Meeting
**Goal:** Bot can join a live Teams meeting via REST API call.

### Prerequisites
- Azure Communication Services resource created (Europe region)
- Azure AD app registered with `Calls.AccessMedia.All` permission
- Teams federation configured via PowerShell (`Set-CsTeamsAcsFederationConfiguration`)
- ngrok or Azure Dev Tunnels running for local development

### Tasks
- [ ] Implement `AcsConfig` with `CallAutomationClient` bean
- [ ] Implement `MeetingJoinService` with `joinMeetingWithAudioStream()`
- [ ] Implement `MeetingController` REST endpoints (join/leave)
- [ ] Implement `AcsCallbackController` for ACS lifecycle events
- [ ] Test: POST meeting link → bot appears in Teams meeting

### Skill Reference
→ `acs-meeting-join`

---

## Phase 3: Audio Stream Reception
**Goal:** Receive and decode real-time audio from ACS via WebSocket.

### Tasks
- [ ] Implement `WebSocketConfig` registering `/ws/audio-stream`
- [ ] Implement `AudioStreamWebSocketHandler` parsing ACS JSON messages
- [ ] Implement `AudioChunkProcessor` for decoded PCM handling
- [ ] Implement `AudioBufferAggregator` for 30-second windowing per speaker
- [ ] Test: join a meeting, speak, verify PCM data arrives and buffers fill

### Skill Reference
→ `audio-websocket-handler`

---

## Phase 4: Speech-to-Text with Whisper
**Goal:** Transcribe buffered audio windows into text using local Whisper.

### Prerequisites
- Whisper installed locally (`brew install whisper-cpp` or `pip install faster-whisper`)
- Whisper server running on `http://localhost:8080`

### Tasks
- [ ] Implement `AudioFormatConverter` (PCM → WAV)
- [ ] Implement `WhisperTranscriptionService` calling local Whisper HTTP API
- [ ] Create `TranscriptEntry` record (speaker, timestamps, text)
- [ ] Wire into `AudioChunkProcessor`: when buffer window flushes → transcribe
- [ ] Test: speak in meeting → see transcript text in logs

### Skill Reference
→ `whisper-transcription`

---

## Phase 5: GDPR Health Data Filter
**Goal:** Filter GDPR Article 9 health information from transcripts before summarization.

### Tasks
- [ ] Implement `RegexHealthFilter` with spoken-language health patterns
- [ ] Implement `GdprFilterService` orchestrating filter pipeline
- [ ] Implement `GdprAuditLogger` for compliance audit trail
- [ ] Implement context-aware filtering (flag surrounding entries)
- [ ] Write comprehensive unit tests (positive + negative cases)
- [ ] (Optional) Implement `LlmHealthClassifier` for second-pass via Ollama
- [ ] Test: inject health-related speech → verify redaction and audit log

### Skill Reference
→ `gdpr-health-filter`

---

## Phase 6: LLM Summarization with Ollama
**Goal:** Produce structured meeting summaries from filtered transcripts.

### Prerequisites
- Ollama installed (`ollama pull llama3`)
- Ollama running on `http://localhost:11434`

### Tasks
- [ ] Implement `OllamaSummaryService` calling Ollama REST API
- [ ] Implement `RollingConversationBuffer` for sliding transcript window
- [ ] Implement `MeetingSummaryService` orchestrating filter → summarize → store
- [ ] Create `MeetingSummary` record with structured sections
- [ ] Configure periodic re-summarization interval
- [ ] Test: filtered transcript → structured summary with topics, decisions, action items

### Skill Reference
→ `ollama-summarization`

---

## Phase 7: Live Frontend Updates
**Goal:** Push real-time summaries to Angular frontend via SSE.

### Tasks
- [ ] Implement `SummaryStreamController` SSE endpoint
- [ ] Implement `SummaryEventPublisher` for event dispatch
- [ ] Define event types: summary-update, transcript-entry, gdpr-redaction, meeting-status
- [ ] Test: summary generated → SSE event pushed → received by client

### Skill Reference
→ `sse-live-updates`

---

## Phase 8: Angular Frontend
**Goal:** Functional UI for joining meetings and viewing live summaries.

### Tasks
- [ ] Create Angular 18+ project
- [ ] Meeting join page with Teams link input
- [ ] Live summary dashboard with SSE subscription
- [ ] GDPR filter statistics display
- [ ] Meeting status indicators
- [ ] Responsive layout

### Skill Reference
→ `angular-frontend`

---

## Phase 9: Integration Testing & Hardening
**Goal:** End-to-end testing and production readiness.

### Tasks
- [ ] End-to-end test: join meeting → speak → see summary in UI
- [ ] Load test: multiple speakers, long meetings
- [ ] GDPR compliance review: verify no health data leaks to LLM or logs
- [ ] Error handling: ACS disconnects, Whisper timeouts, Ollama failures
- [ ] Security review: secrets management, input validation
- [ ] Documentation: README, API docs, deployment guide