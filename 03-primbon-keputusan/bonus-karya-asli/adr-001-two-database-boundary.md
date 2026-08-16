---
aliases:
  - "Chatbot Kasir - ADR-001: Two-Database Boundary"
---

# Chatbot Kasir - ADR-001: Two-Database Boundary

Related: [Chatbot Kasir - Architecture Decisions](../../02-PROJECTS/chatbot-kasir/02-architecture-decisions.md), [Chatbot Kasir - Project Identity](../../02-PROJECTS/chatbot-kasir/01-project-identity.md), [Chatbot Kasir - Codebase Map](../../02-PROJECTS/chatbot-kasir/04-codebase-map.md)

## Status

Accepted — 2026-05-30 (formalized from early project decision)

## Context

The chatbot needs to read product master data (catalog, stock) from the main POS application, while also writing chat staging records (sessions, messages, drafts, outbox, ledger). Both stores live in PostgreSQL. The question: one shared database, or two?

## Decision

The service connects to **two PostgreSQL databases**:

- `pos_system` (product database) — strictly **read-only** from this service. Migrated from the main app, not from this repo. Alembic does not target it. ORM mapping in `app/infrastructure/db/product/models.py` is read-only.
- `ai_chat_mvp` (chat database) — **read-write** owned by this service. Alembic migrations originate here.

## Rationale

- Main POS app already owns product master data. Two-way sync from the chatbot would introduce write conflicts, drift, and audit complexity.
- AI is fundamentally probabilistic. Letting it write to product master would risk corrupting the source of truth.
- Chat-side staging is conceptually different data — conversations, AI interpretations, draft state — and benefits from a separate schema lifecycle.

## Alternatives Considered

- **Single database with strict permission scoping** — rejected. Permission boundaries inside one DB are weaker than physical separation and harder to audit.
- **ORM read-replicas** — rejected. Adds infra complexity without removing the conceptual blur between "main app data" and "chat staging data".

## Forward Implication

- New tables for Kasir domain ALWAYS land in chat DB. Never extend product DB schema from this repo.
- Any time AI suggests writing to `product_product`, `product_stock`, etc., that suggestion is wrong. Push back.
- Alembic only targets chat DB.

## Implementation Pointer

- `app/infrastructure/db/chat_postgres.py` — chat DB session factory
- `app/infrastructure/db/product_postgres.py` — product DB session factory (read-only)
- `app/infrastructure/db/product/models.py` — product DB ORM (read-only)
- `app/infrastructure/db/models.py` — chat DB ORM (read-write)
- `alembic/env.py` — targets chat DB only
