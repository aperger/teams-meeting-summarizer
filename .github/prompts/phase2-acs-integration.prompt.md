# Phase 2: ACS Call Automation – Join Teams Meeting

## Context
Read SKILLS.md (skill: acs-meeting-join), DEVELOPMENT_PHASES.md (Phase 2),
and .github/copilot-instructions.md for full context.

## Task
Implement the Azure Communication Services integration to join Teams meetings:

1. `AcsConfig` — Bean definition for `CallAutomationClient` using connection string from config
2. `MeetingJoinService` — Joins a Teams meeting using `TeamsMeetingLinkLocator`,
   configures `MediaStreamingOptions` for:
   - WebSocket transport to `{callback-base-url}/ws/audio-stream`
   - PCM 16kHz mono audio format
   - Unmixed audio channel (per-speaker)
   - Start streaming immediately on join
3. `MeetingController` — REST endpoints:
   - `POST /api/meetings/join` — accepts `{ "teamsMeetingLink": "..." }`, returns connection ID
   - `POST /api/meetings/{connectionId}/leave` — leaves the meeting
4. `AcsCallbackController` — `POST /api/callbacks` handling:
   - `CallConnected` — log join success
   - `CallDisconnected` — log disconnect
   - `ParticipantsUpdated` — track speakers

## Constraints
- Use record types for request/response DTOs
- All errors return ProblemDetail (RFC 7807)
- Connection string from environment variable, never hardcoded
- Follow copilot-instructions.md strictly