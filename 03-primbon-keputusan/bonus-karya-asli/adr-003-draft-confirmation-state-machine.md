---
aliases:
  - "Chatbot Kasir - ADR-003: Draft + Confirmation State Machine"
---

# Chatbot Kasir - ADR-003: Draft + Confirmation State Machine

Related: [Chatbot Kasir - Architecture Decisions](../../02-PROJECTS/chatbot-kasir/02-architecture-decisions.md), [Chatbot Kasir - ADR-002: AI Only Interprets, Business Engine Executes](adr-002-ai-only-interprets.md), [Chatbot Kasir - Domain Glossary](../../02-PROJECTS/chatbot-kasir/03-domain-glossary.md)

## Status

Accepted — 2026-05-30 (formalized from Phase K6/K7 implementation)

## Context

If AI interpretations can be wrong, and business mutations must be deterministic and explicit, every mutation needs a clear "pending → confirmed → committed" lifecycle that the merchant participates in.

## Decision

Every operational mutation transitions through this state machine:

```
needs_clarification  ← AI needs more info
       ↓
pending_confirmation ← AI parsed enough, waiting on merchant "iya"
       ↓
   confirmed         ← merchant approved
       ↓
   outbox + ledger written
       ↓
   later sync to main app (separate process)
```

Cancel and update transitions are also explicit states (`cancelled`, mutation re-routes back to `pending_confirmation`). Additional terminal states: `expired`, `failed`.

## Rationale

- Forces every mutation to have an explicit merchant approval point.
- Allows updates (corrections like "bukan 3000, 4000") to mutate the draft instead of creating phantom transactions.
- Allows cancel to leave a deterministic trail (via `draft_events` table).
- Avoids hidden state — every transition writes to `draft_events`.

## Forward Implication

- Worker dispatch (`_handle_operational_message`) is the orchestrator that wires interpretation → draft state transitions.
- No mutation lands in `operational_command_outbox` or `temporary_ledger_entries` without `draft.status == "confirmed"`.
- Active draft (status `needs_clarification` or `pending_confirmation`) influences how the next message is interpreted.

## Implementation Pointer

- `app/domain/operational/service.py` — state transition logic
- `app/cognition/worker.py:_handle_operational_message` — dispatch orchestrator
- `app/infrastructure/db/models.py` — `OperationalDraft`, `DraftEvent`, `DraftConfirmation` tables
- Draft state values defined in `app/cognition/operational/schemas.py` `DraftState`
