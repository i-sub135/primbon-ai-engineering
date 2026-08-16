---
aliases:
  - "Chatbot Kasir - ADR-012: Render Fallbacks Must Not Lie About Persistence"
---

# Chatbot Kasir - ADR-012: Render Fallbacks Must Not Lie About Persistence

Related: [Chatbot Kasir - Architecture Decisions](../../02-PROJECTS/chatbot-kasir/02-architecture-decisions.md), [Chatbot Kasir - ADR-010: Cancellation Phrases Always Bypass Hijack](adr-010-cancellation-phrases-bypass-hijack.md), [Chatbot Kasir - Known Pitfalls](../../02-PROJECTS/chatbot-kasir/07-known-pitfalls.md)

## Status

Accepted — 2026-05-29 (commit `6becacd`, fix/render-fallback)

## Context

A merchant typing `"iya"` out of context, OR a candidate-selection hijack that failed to recover, used to fall through to a literal `"Data tersimpan."` response — even though nothing was saved. This is dangerous: merchant believes their transaction is in the books when it is not.

QA caught this both in:
- Stale CONFIRM_DRAFT fallback path (no draft, no active_draft)
- "batal" hijacked into candidate-selection then bouncing to render (now also fixed via ADR-010)

## Decision

`_render_operational_response` MUST NOT return `"Data tersimpan."` when no draft was actually saved.

- CONFIRM_DRAFT path with no draft + no active_draft returns `"Belum ada transaksi yang bisa disimpan."`
- CANCEL_DRAFT path with no draft + no active_draft returns `"Oke, tidak jadi."`
- Successful confirmation (draft.status == "confirmed") still returns `"Data tersimpan."` — the message is now reserved for actual success.

## Rationale

- Truth in UX. Merchant must not be misled about persistence state.
- Single deceptive response erodes trust faster than many minor UX issues.
- The fallback wording is friendly enough to not feel like an error message.

## Forward Implication

- Any new fallback wording must explicitly check the persistence state. Never claim a save without `draft.status == "confirmed"` or equivalent verification.
- Adding new intents must include explicit handling of no-state fallbacks.

## Implementation Pointer

- `app/cognition/worker.py:_render_operational_response` — the renderer
- Tests in `tests/test_worker_operational_response.py` — covers no-draft fallback assertions
