# Phase 6: Ollama LLM Summarization

## Context
Read SKILLS.md (skill: ollama-summarization), DEVELOPMENT_PHASES.md (Phase 6),
and .github/copilot-instructions.md for full context.

## Task
Integrate local Ollama for meeting summarization:

1. `OllamaSummaryService` — calls Ollama `POST /api/generate` with:
   - Model from config (default: `llama3`)
   - `stream: false` for complete responses
   - Structured summarization prompt (see prompt requirements in SKILLS.md)
2. `RollingConversationBuffer` — sliding window of filtered `TranscriptEntry` items,
   configurable time window (default: last 30 minutes)
3. `MeetingSummaryService` — orchestrates: buffer → filter check → summarize → store
   Triggered periodically (configurable, default every 5 minutes)
4. `MeetingSummary` record — structured output:
   - `(String fullSummary, List<String> topicsDiscussed, List<ActionItem> actionItems, List<String> decisions, List<String> openQuestions, long redactedCount, Instant generatedAt)`
5. `ActionItem` record — `(String description, String owner, String deadline)`

## Prompt Engineering
The prompt must instruct the LLM to output in a parseable structure.
Include explicit instructions to NEVER include health/medical/wellbeing information.

## Constraints
- Use Spring `WebClient` for Ollama HTTP calls
- Proper timeout handling (LLM inference can be slow, especially on CPU)
- Unit tests with mocked Ollama responses
- Follow copilot-instructions.md strictly