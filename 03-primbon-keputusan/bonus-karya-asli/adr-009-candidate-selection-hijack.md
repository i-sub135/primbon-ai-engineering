---
aliases:
  - "Chatbot Kasir - ADR-009: Candidate-Selection Hijack For Sale Ambiguity Replies"
---

# Chatbot Kasir - ADR-009: Candidate-Selection Hijack For Sale Ambiguity Replies

Related: [Chatbot Kasir - Architecture Decisions](../../02-PROJECTS/chatbot-kasir/02-architecture-decisions.md), [Chatbot Kasir - ADR-010: Cancellation Phrases Always Bypass Hijack](adr-010-cancellation-phrases-bypass-hijack.md), [Chatbot Kasir - Known Pitfalls](../../02-PROJECTS/chatbot-kasir/07-known-pitfalls.md), [Chatbot Kasir - Current State](../../02-PROJECTS/chatbot-kasir/05-current-state.md)

## Status

Accepted — 2026-05-29 (commit `96c56ea`, expanded from earlier `f83166d`)

## Context

After "jual minyak" → "1. Minyak Angin / 2. Minyak Tanah / 3. Minyak Goreng / Sekalian sebutkan jumlahnya", the merchant typically replies with "minyak goreng 3", "1", "minyak tanah", or similar. Sending this to the LLM is wasteful and error-prone — it's a structured deterministic selection.

## Decision

When the previous interpretation was `CREATE_SALE + requires_clarification` with product candidates, AND the next message looks like a short reply (`_looks_like_candidate_selection`), the worker bypasses LLM and builds a synthetic `CONFIRM_DRAFT` interpretation that triggers `_recover_sale_from_candidate_selection`.

The recovery function extracts trailing quantity from selection_text if the prior interpretation lacked qty.

## Rationale

- Deterministic > probabilistic for known-structured input.
- AI calls cost latency and money; bypass when structure is clear.
- Recovery uses existing candidate list, so output matches what merchant saw.

## Forward Implication & Known Bug

- The hijack interacts with the active-draft state machine in a way that currently causes Finding 6 (silent wrong-data persistence). See [Chatbot Kasir - Known Pitfalls](../../02-PROJECTS/chatbot-kasir/07-known-pitfalls.md) and QA report Finding 6 in source repo.
- Phase K19 fixes this interaction. Until then, the hijack can confirm the WRONG draft when ambiguity context AND pending-confirmation draft coexist.

## Implementation Pointer

- `app/cognition/worker.py:_looks_like_candidate_selection` — heuristic
- `app/cognition/worker.py:_has_pending_sale_candidate_context` — guard
- `app/cognition/worker.py:_build_candidate_selection_interpretation` — synthetic CONFIRM_DRAFT
- `app/cognition/worker.py:_recover_sale_from_candidate_selection` — recovery + qty extraction
- Tests in `tests/test_worker_operational_response.py`
