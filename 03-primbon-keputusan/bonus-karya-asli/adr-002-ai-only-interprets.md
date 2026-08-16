---
aliases:
  - "Chatbot Kasir - ADR-002: AI Only Interprets, Business Engine Executes"
---

# Chatbot Kasir - ADR-002: AI Only Interprets, Business Engine Executes

Related: [Chatbot Kasir - Architecture Decisions](../../02-PROJECTS/chatbot-kasir/02-architecture-decisions.md), [Chatbot Kasir - Project Identity](../../02-PROJECTS/chatbot-kasir/01-project-identity.md), [Chatbot Kasir - ADR-003: Draft + Confirmation State Machine](adr-003-draft-confirmation-state-machine.md)

## Status

Accepted — 2026-05-30 (formalized from early project decision, foundational non-negotiable rule)

## Context

The system needs to translate messy merchant chat ("jual 2 minyak goreng cash") into business mutations (sale recorded, ledger written, stock decremented in the main app). The question: how much can the AI directly do?

## Decision

The AI layer (LLM + LangGraph cognition workflow) produces **structured interpretation output** (Pydantic-validated JSON conforming to `InterpretationResult`). It never directly creates a final transaction, ledger entry, or product mutation.

A separate deterministic **business engine** (the operational domain services in `app/domain/operational/`) consumes the interpretation, creates a draft, awaits explicit merchant confirmation, then writes outbox + ledger.

## Rationale

- LLMs hallucinate. Even high-quality interpretations occasionally pick wrong product, wrong amount, wrong intent. A draft + confirmation gate gives the merchant a chance to catch errors before they hit the books.
- Operational finance demands deterministic, auditable, repeatable mutations. The same input → same output. LLMs cannot guarantee this; deterministic services can.
- The draft-confirmation flow doubles as a UX gate: merchant sees what the AI captured before agreeing. Transparency builds trust.

## Alternatives Considered

- **Direct AI-to-ledger commit** — rejected on safety grounds.
- **AI as final authority, business engine as validator only** — rejected; AI sometimes generates plausible but wrong structured output. The business engine must own the final write decision.

## Forward Implication

- Every mutation in chat DB that affects the merchant's books must come from a confirmed draft. Out-of-band writes are bugs.
- The interpretation service's job ends when it returns an `InterpretationResult` plus persisted `ai_interpretations` row. The draft service takes over from there.
- AI suggestions to "just write directly" must be refused.

## Implementation Pointer

- `app/cognition/operational/service.py` — `OperationalInterpretationService` (the AI-facing interpretation service)
- `app/domain/operational/service.py` — `OperationalDraftService` (the deterministic business engine)
- `app/cognition/operational/schemas.py` — `InterpretationResult` Pydantic schema
