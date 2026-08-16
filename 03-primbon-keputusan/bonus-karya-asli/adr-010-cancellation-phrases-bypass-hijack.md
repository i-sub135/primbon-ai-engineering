---
aliases:
  - "Chatbot Kasir - ADR-010: Cancellation Phrases Always Bypass Hijack"
---

# Chatbot Kasir - ADR-010: Cancellation Phrases Always Bypass Hijack

Related: [Chatbot Kasir - Architecture Decisions](../../02-PROJECTS/chatbot-kasir/02-architecture-decisions.md), [Chatbot Kasir - ADR-009: Candidate-Selection Hijack For Sale Ambiguity Replies](adr-009-candidate-selection-hijack.md), [Chatbot Kasir - ADR-012: Render Fallbacks Must Not Lie About Persistence](adr-012-render-fallbacks-must-not-lie.md)

## Status

Accepted — 2026-05-29 (commit `6becacd`, fix/render-fallback)

## Context

Without an explicit block list, "batal" after a clarification was being hijacked into the candidate-selection path. The recovery would fail (because the prior interpretation lacked quantity), the dispatch would fall through, and the render would say "Data tersimpan." — claiming a save when nothing was saved. QA caught this.

## Decision

`_looks_like_candidate_selection` returns `False` for cancellation phrases: `batal`, `cancel`, `jangan`, `hapus`, `ulang`, `salah`, `gak jadi`, `ga jadi`, `nggak jadi`, `tidak jadi`.

These phrases ALWAYS reach the cancel fast path, never get wrapped as synthetic CONFIRM_DRAFT.

## Rationale

- Cancellation intent is unambiguous in Indonesian merchant context.
- Misclassifying cancel as candidate-selection is high-cost (false claim of save).
- Block list is small and stable; cancellation vocabulary in Indonesian is well-defined.

## Forward Implication

- New cancellation synonyms must be added to BOTH:
  - `_CANDIDATE_SELECTION_BLOCK_PHRASES` in `app/cognition/worker.py`
  - `_CANCELLATION_PHRASES` in `app/cognition/operational/service.py`
- Keeping these in sync is part of the operational rules; missing one leads to subtle dispatch bugs.

## Implementation Pointer

- `app/cognition/worker.py:_CANDIDATE_SELECTION_BLOCK_PHRASES` — the block set
- `app/cognition/worker.py:_looks_like_candidate_selection` — block check happens first
- Tests in `tests/test_worker_operational_response.py`
