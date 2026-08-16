---
aliases:
  - "Chatbot Kasir - ADR-007: Operational Intent Schema Owned By This Repo"
---

# Chatbot Kasir - ADR-007: Operational Intent Schema Owned By This Repo

Related: [Chatbot Kasir - Architecture Decisions](../../02-PROJECTS/chatbot-kasir/02-architecture-decisions.md), [Chatbot Kasir - Domain Glossary](../../02-PROJECTS/chatbot-kasir/03-domain-glossary.md)

## Status

Accepted — 2026-05-30 (formalized from Phase K2 implementation)

## Context

The chatbot translates messy merchant chat into structured intents (`CREATE_SALE`, `QUERY_STOCK`, etc.). The main POS app has its own command vocabulary on the other side of the eventual sync. Question: shared schema, or independent?

## Decision

The operational intent vocabulary is defined and validated inside this repo, NOT shared with the main app. The canonical enum lives in `app/cognition/operational/schemas.py`.

## Rationale

- The chatbot translates messy merchant chat into clean intents. The main app already has its own command vocabulary on the other side of the eventual outbox sync.
- Coupling intent vocabulary across two products would slow iteration.
- The chatbot's intents are AI-shaped (they have `requires_clarification`, `candidate_values`, etc.) — the main app's commands are deterministic-shaped. Different concerns.

## Forward Implication

- New intents added in this repo only (Phase K10 expansion added `QUERY_TRANSACTIONS` and `QUERY_PRODUCTS` on 2026-05-29).
- Outbox sync layer (future cronjob) translates chatbot intent vocabulary → main app command vocabulary.
- Schema validation via Pydantic prevents malformed AI output from reaching the draft service.

## Implementation Pointer

- `app/cognition/operational/schemas.py` — `OperationalIntent` Literal type
- `docs/V2/operational-intent-schema.md` — spec doc in source repo
- Adding a new intent requires: enum update + fast-path or AI prompt update + worker handler
