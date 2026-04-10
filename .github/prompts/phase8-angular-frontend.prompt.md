# Phase 8: Angular Frontend

## Context
Read SKILLS.md (skill: angular-frontend), DEVELOPMENT_PHASES.md (Phase 8),
and .github/copilot-instructions.md for full context.

## Task
Build the Angular 18+ frontend application:

1. **Meeting Join Page**
   - Text input for Teams meeting join link
   - "Join Meeting" button → POST to `/api/meetings/join`
   - "Leave Meeting" button → POST to `/api/meetings/{id}/leave`
   - Status indicator (disconnected / joining / connected / error)

2. **Live Summary Dashboard**
   - Subscribe to SSE at `/api/stream/summary/{connectionId}`
   - Display structured summary sections:
     - Topics Discussed
     - Decisions Made
     - Action Items (with owner and deadline)
     - Open Questions
   - Auto-updates when new `summary-update` events arrive

3. **GDPR Filter Statistics**
   - Display count of redacted messages
   - Visual indicator when redaction occurs

4. **Meeting Status Bar**
   - Current participants count
   - Meeting duration timer
   - Connection status

## Constraints
- Angular 18+ with standalone components
- Responsive layout (works on desktop and tablet)
- Use Angular HttpClient for REST calls
- Use EventSource API for SSE subscription
- No UI framework required — keep it simple for PoC