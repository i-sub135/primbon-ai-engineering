---
aliases:
  - "Chatbot Kasir - ADR-004: Channel-Agnostic Chat Domain"
---

# Chatbot Kasir - ADR-004: Channel-Agnostic Chat Domain

Related: [Chatbot Kasir - Architecture Decisions](../../02-PROJECTS/chatbot-kasir/02-architecture-decisions.md), [Chatbot Kasir - Project Identity](../../02-PROJECTS/chatbot-kasir/01-project-identity.md), [Chatbot Kasir - Codebase Map](../../02-PROJECTS/chatbot-kasir/04-codebase-map.md)

## Status

Accepted — 2026-05-30 (formalized from Phase 6B implementation)

## Context

The project starts with web chat (FastAPI HTTP + SSE) but is designed for future WhatsApp expansion. Future channels (Telegram, Line, etc.) should be addable without touching cognition or domain code.

## Decision

The chat domain (`app/domain/chat/`) does NOT know about WhatsApp, web, Slack, or any specific channel. Provider-specific code lives under `app/infrastructure/channels/`. The cognition layer NEVER imports WhatsApp clients.

## Rationale

- Coupling cognition to a specific channel would force rewrites later.
- Channel adapters can be added/removed without touching domain logic.
- Easier to test the cognition flow against a fake channel.
- WhatsApp adapter currently sits as a scaffold; full integration is on hold pending WhatsApp Business API infra access.

## Forward Implication

- WhatsApp is treated as a future channel adapter, not a core feature.
- Any time AI suggests importing WhatsApp specifics inside `app/cognition/`, that's a violation. Push back.
- Same applies to web-specific imports (FastAPI request/response objects) — cognition stays HTTP-independent.

## Implementation Pointer

- `app/domain/chat/service.py` — `ChatService` is channel-agnostic
- `app/infrastructure/channels/` — channel adapter scaffolds live here
- Import boundary check is part of repo conventions; see [Chatbot Kasir - Operational Rules](../../02-PROJECTS/chatbot-kasir/06-operational-rules.md)
