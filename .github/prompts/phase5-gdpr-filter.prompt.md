# Phase 5: GDPR Article 9 Health Data Filter

## Context
Read SKILLS.md (skill: gdpr-health-filter), DEVELOPMENT_PHASES.md (Phase 5),
and .github/copilot-instructions.md for full context.

## Task
Implement a two-pass GDPR filter for spoken language transcripts:

1. `RegexHealthFilter` — pattern-based detection covering:
   - Direct medical terms (diagnosis, medication, surgery, hospital, therapy, cancer, diabetes,
     allergy, depression, anxiety, pregnancy, chemotherapy, physiotherapy, etc.)
   - Spoken health phrases ("feeling unwell", "called in sick", "recovery took",
     "on medication", "mental health", "burnout", "stress leave", "back from hospital", etc.)
   - ICD-10 codes (e.g., A00-Z99 pattern)
2. `GdprFilterService` — orchestrates filtering pipeline:
   - First pass: regex
   - Context-aware: if entry is flagged, also flag next 1-2 entries
   - Optional second pass: LLM classification via Ollama
3. `LlmHealthClassifier` — asks Ollama to classify borderline text as health-related or not
4. `FilterResult` record — `(TranscriptEntry original, boolean redacted, String outputText, List<String> matchedTerms, String reason)`
5. `GdprAuditLogger` — logs filter decisions (timestamp, speaker, reason, matched terms)
   but NEVER logs the redacted content itself

## Critical Rules
- Prefer over-filtering to under-filtering (false positives are acceptable, false negatives are not)
- Redacted content must NEVER appear in logs, LLM prompts, or API responses
- Comprehensive unit tests required: health terms, health phrases, ICD codes, clean text, edge cases

## Constraints
- Follow copilot-instructions.md strictly