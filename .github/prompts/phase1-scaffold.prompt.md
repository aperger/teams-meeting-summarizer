# Phase 1: Scaffold Spring Boot 4.x Project

## Context
Read SKILLS.md (skill: scaffold-spring-boot-project), DEVELOPMENT_PHASES.md (Phase 1),
and .github/copilot-instructions.md for full context.

## Task
Create the initial Spring Boot 4.0.x Maven project:

1. Generate `pom.xml` with:
   - Spring Boot 4.0.5 parent
   - Java 21 source/target
   - All dependencies listed in SKILLS.md scaffold skill
   - Spring Cloud Azure 7.1.0 BOM
   - Maven wrapper

2. Create `application.yml` with placeholder sections for:
   - `acs.connection-string`, `acs.callback-base-url`
   - `whisper.endpoint`, `whisper.model`, `whisper.language`
   - `ollama.base-url`, `ollama.model`, `ollama.summary-interval-seconds`
   - `azure.tenant-id`, `azure.client-id`, `azure.client-secret`
   - All values referencing `${ENV_VAR}` placeholders

3. Create empty package structure under `com.summarizer.teams`:
   - config, controller, service.acs, service.audio, service.transcription,
     service.gdpr, service.summary, model, websocket, exception

4. Create a `GlobalExceptionHandler` with `@RestControllerAdvice` returning `ProblemDetail`

5. Create main application class

## Constraints
- Follow all rules in copilot-instructions.md
- Jakarta EE 11 imports only (`jakarta.*`)
- Constructor injection only
- No source code for business logic — only skeleton/structure