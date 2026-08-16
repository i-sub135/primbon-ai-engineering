---
aliases:
  - "Chatbot Kasir - ADR-005: Redis Streams for Async Boundary"
---

# Chatbot Kasir - ADR-005: Redis Streams for Async Boundary

Related: [Chatbot Kasir - Architecture Decisions](../../02-PROJECTS/chatbot-kasir/02-architecture-decisions.md), [Chatbot Kasir - Codebase Map](../../02-PROJECTS/chatbot-kasir/04-codebase-map.md)

## Status

Accepted — 2026-05-30 (formalized from Phase 5A2/5A3 implementation)

## Context

The API needs to enqueue AI work without blocking the HTTP response. The cognition worker needs to consume jobs durably and stream tokens back to the client. Redis Pub/Sub vs Redis Streams was the choice.

## Decision

The async boundary between API (writer) and worker (consumer) uses **Redis Streams** with consumer groups, NOT Redis Pub/Sub.

## Rationale

- Streams give us durable messaging with replay capability, which Pub/Sub does not.
- Consumer groups let us scale workers horizontally with at-most-once delivery semantics.
- Stream IDs serve as natural job correlation keys.

## Alternatives Considered

- **Redis Pub/Sub** — rejected. Fire-and-forget; messages lost if no consumer is connected.
- **PostgreSQL-based queue** — rejected for now; adds load to the chat DB and requires extra mechanism for stream/SSE behaviour.
- **RabbitMQ / Kafka** — rejected as overkill for MVP scope.

## Forward Implication

- The job queue (`REDIS_AI_JOB_STREAM`) is a Stream, not a channel.
- SSE streaming to the client uses a separate per-request Stream (`stream:{request_id}`).
- Worker scaling = add more consumers to the consumer group.
- Production hardening (e.g., XCLAIM for stuck messages) belongs in the polishing phase.

## Implementation Pointer

- `app/infrastructure/redis/` — stream consumer + producer wrappers
- `app/cognition/worker.py` — consumes from `REDIS_AI_JOB_STREAM`
- `app/api/routes/stream.py` — SSE endpoint reads from `stream:{request_id}`
