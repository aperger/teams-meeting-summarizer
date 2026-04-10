# Phase 7: Server-Sent Events for Live Updates

## Context
Read SKILLS.md (skill: sse-live-updates), DEVELOPMENT_PHASES.md (Phase 7),
and .github/copilot-instructions.md for full context.

## Task
Implement SSE endpoint to push live updates to the Angular frontend:

1. `SummaryStreamController` — `GET /api/stream/summary/{connectionId}`
   Returns `SseEmitter` streaming events to the client
2. `SummaryEventPublisher` — Spring `ApplicationEventPublisher` wrapper that
   publishes events when summaries are generated or meeting status changes
3. Event types:
   - `summary-update` — new or updated MeetingSummary
   - `transcript-entry` — new (filtered) transcript line
   - `gdpr-redaction` — notification that content was redacted (count only, no content)
   - `meeting-status` — joined, participant-changed, disconnected

## Constraints
- Handle client disconnect gracefully (remove emitter)
- Set appropriate SSE timeout (default: 30 minutes for long meetings)
- Follow copilot-instructions.md strictly