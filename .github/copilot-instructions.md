# Copilot Instructions – Teams Meeting Summarizer

## Project Overview
This is a **Teams Live Meeting Summarizer** — a Spring Boot 4.x backend + Angular frontend application.
It joins Microsoft Teams meetings via Azure Communication Services (ACS), captures real-time audio streams,
transcribes speech using local Whisper, filters out GDPR Article 9 health data, and produces rolling
meeting summaries using a local Ollama LLM.

## Tech Stack (Strict)
- **Java 21** (minimum, baseline for Spring Boot 4.x / Spring Framework 7 / Jakarta EE 11)
- **Spring Boot 4.0.x** (latest 4.0.5+) — do NOT use Spring Boot 3.x or earlier
- **Spring Framework 7.x** — comes with Spring Boot 4.x
- **Jakarta EE 11** — all imports must use `jakarta.*`, never `javax.*`
- **Azure Communication Services Java SDK** (`azure-communication-callautomation` 1.6.x)
- **Spring Cloud Azure 7.x** for Azure integration
- **Angular 18+** for frontend
- **Ollama** (local LLM, REST API at `http://localhost:11434`)
- **Whisper.cpp** or **faster-whisper** (local Speech-to-Text)
- **Maven** as build tool (not Gradle)
- **JUnit 5 + Mockito** for testing

## Architecture Principles
- Modular layered architecture: Controller → Service → Component
- Each concern in its own service: audio capture, transcription, GDPR filtering, summarization
- WebSocket for receiving ACS audio streams
- REST API for Angular frontend communication
- Server-Sent Events (SSE) for pushing live summaries to frontend
- All secrets via environment variables or Spring Cloud Azure Key Vault — never hardcoded
- GDPR-by-design: filter health data BEFORE it reaches the LLM

## Coding Standards
- Use Java 21 features: records, sealed classes, pattern matching, text blocks, virtual threads where appropriate
- Use `jakarta.*` imports exclusively — never `javax.*`
- Prefer constructor injection (no field injection with `@Autowired`)
- Use `record` types for DTOs, request/response objects, and value objects
- Use `sealed interface` + `permits` for domain event hierarchies
- Use `Optional` return types instead of returning null
- Use `java.time` API exclusively — never `java.util.Date`
- Use SLF4J (`@Slf4j` Lombok annotation) for logging
- All public service methods must have Javadoc
- Max method length: 30 lines — extract helper methods if longer
- No wildcard imports
- Use `final` for local variables that are not reassigned

## Package Structure
```
com.summarizer.teams
├── config/              # Spring configuration classes
├── controller/          # REST and WebSocket controllers
├── service/
│   ├── acs/             # ACS call automation, meeting join
│   ├── audio/           # Audio stream processing, buffering
│   ├── transcription/   # Whisper STT integration
│   ├── gdpr/            # GDPR health data filtering
│   └── summary/         # Ollama LLM summarization
├── model/               # Domain records, DTOs, events
├── websocket/           # WebSocket handlers for ACS audio
└── exception/           # Custom exceptions and error handling
```

## Naming Conventions
- Service classes: `*Service` (e.g., `MeetingJoinService`, `GdprFilterService`)
- Configuration classes: `*Config` (e.g., `AcsConfig`, `WebSocketConfig`)
- Controllers: `*Controller` (e.g., `MeetingController`)
- WebSocket handlers: `*WebSocketHandler` (e.g., `AudioStreamWebSocketHandler`)
- Records for DTOs: descriptive name without suffix (e.g., `JoinRequest`, `MeetingSummary`)
- Test classes: `*Test` for unit, `*IntegrationTest` for integration tests

## Error Handling
- Use `@RestControllerAdvice` with `@ExceptionHandler` for global exception handling
- Define custom exceptions extending `RuntimeException` in the `exception` package
- Always return `ProblemDetail` (RFC 7807) responses for errors
- Log exceptions at the appropriate level (ERROR for unexpected, WARN for business rule violations)

## Security & GDPR Rules
- NEVER send unfiltered transcript text to the LLM
- ALWAYS apply GDPR health data filter before any summarization
- Log what was filtered and why (audit trail) — but never log the redacted content itself
- All ACS connection strings and Azure credentials via environment variables
- Use `@Value("${...}")` referencing env vars, not hardcoded values

## Testing
- Unit test every service class
- Mock external dependencies (ACS SDK, Ollama HTTP client, Whisper process)
- Integration tests for WebSocket audio reception
- Test GDPR filter with both positive (should redact) and negative (should pass) cases
- Minimum 80% code coverage target

## What NOT To Do
- Do NOT use Spring Boot 3.x or earlier APIs
- Do NOT use `javax.*` packages — use `jakarta.*`
- Do NOT hardcode secrets, connection strings, or API keys
- Do NOT send unfiltered text to Ollama
- Do NOT use `@Autowired` on fields — use constructor injection
- Do NOT create God classes — keep single responsibility
- Do NOT use `Thread.sleep()` — use virtual threads or reactive patterns
- Do NOT return `null` from service methods — use `Optional` or throw exceptions