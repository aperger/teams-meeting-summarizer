# AGENTS.md

## Mission and Current State
- This repo targets a Teams live-meeting summarizer pipeline: ACS join -> audio ingest -> Whisper STT -> GDPR filter -> Ollama summary -> Angular UI (`README.md`, `SKILLS.md`).
- Current code is a scaffold, not the full pipeline: only app bootstrap + context test exist in `app/src/main/java/com/summarizer/teams/TeamsSummarizerApplication.java` and `app/src/test/java/com/summarizer/teams/TeamsSummarizerApplicationTests.java`.
- Treat `SKILLS.md` + `DEVELOPMENT_PHASES.md` as the executable roadmap; each phase has a matching prompt in `.github/prompts/`.

## Architecture You Should Preserve
- Intended backend shape is layered and modular: controller -> service -> component, with services split by concern (`.github/copilot-instructions.md`).
- Planned transport boundaries are explicit: REST join/leave, WebSocket audio ingest, SSE summary fanout.
- Canonical endpoint contracts to keep stable:
  - `POST /api/meetings/join`, `POST /api/meetings/{connectionId}/leave`, `POST /api/callbacks` (`SKILLS.md`, `phase2-acs-integration.prompt.md`).
  - `/ws/audio-stream` for ACS media (`phase3-audio-websocket.prompt.md`).
  - `GET /api/stream/summary/{connectionId}` for SSE (`phase7-sse-updates.prompt.md`).
- GDPR rule is non-negotiable: filter transcript content before LLM summarization (`README.md`, `.github/copilot-instructions.md`, `SKILLS.md`).

## Codebase-Specific Conventions (Important)
- Use Spring Boot `4.0.5` + Java `21` as already pinned in `pom.xml`; avoid Boot 3 APIs.
- Use `jakarta.*` (never `javax.*`) per `.github/copilot-instructions.md`.
- Prefer constructor injection; use records for DTO/value types; return `ProblemDetail` for API errors.
- Backend is now split into Maven modules:
  - `app`: Spring Boot entrypoint and web/API layer.
  - `infra`: external integration adapters.
  - `model`: shared domain records/DTOs.
- Use `com.summarizer.teams` as the base package for new code.

## Developer Workflows That Matter
- Build and test with Maven wrapper from repo root:
  - `./mvnw clean compile`
  - `./mvnw test`
  - `./mvnw spring-boot:run`
- Current `application.yaml` only sets app name; expected ACS/Whisper/Ollama/Azure config placeholders are documented but not yet added (`DEVELOPMENT_PHASES.md`, `phase1-scaffold.prompt.md`).
- External runtime dependencies assumed by design docs:
  - Whisper HTTP service (default shown as `http://localhost:8080/inference`).
  - Ollama API (`http://localhost:11434`).
  - Public callback/WebSocket URL for ACS during local dev (ngrok/dev tunnel in `README.md`).

## How AI Agents Should Execute Work
- Implement by phase, one vertical slice at a time, using `.github/prompts/phase*.prompt.md` as task contracts.
- Ground decisions in existing docs first, then code reality: do not claim a feature exists unless implemented under `src/main`.
- Keep integrations mockable (ACS SDK, Whisper, Ollama) and add tests as each service appears (`SKILLS.md` acceptance criteria).
- Avoid hardcoded secrets; wire env-driven config keys before integration logic (`README.md`, `.github/copilot-instructions.md`).

