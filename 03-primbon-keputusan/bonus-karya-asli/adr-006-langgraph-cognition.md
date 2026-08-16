---
aliases:
  - "Chatbot Kasir - ADR-006: LangGraph for Cognition Workflow Orchestration"
---

# Chatbot Kasir - ADR-006: LangGraph for Cognition Workflow Orchestration

Related: [Chatbot Kasir - Architecture Decisions](../../02-PROJECTS/chatbot-kasir/02-architecture-decisions.md), [Chatbot Kasir - Codebase Map](../../02-PROJECTS/chatbot-kasir/04-codebase-map.md)

## Status

Accepted — 2026-05-30 (formalized from Phase 5A5 implementation)

## Context

The cognition layer needs to orchestrate multiple steps (intent classification, routing memory lookup, LLM interpretation, memory writeback) per merchant message, with state that persists across messages within a session.

## Decision

The cognition worker uses LangGraph to orchestrate interpretation, routing, memory access, and LLM calls. LangGraph state is checkpointed to PostgreSQL (`ai_langgraph_checkpoints` table in chat DB).

## Rationale

- LangGraph gives us explicit nodes and edges for the cognition flow, making reasoning paths inspectable.
- Checkpointing to Postgres preserves conversation state across worker restarts.
- The graph layer separates orchestration from prompt logic, letting prompt work happen independently.
- Provider-agnostic LLM client wraps the underlying provider (OpenAI / Ollama / Anthropic), so model switching does not require graph rewiring.

## Forward Implication

- Cognition graph nodes live in `app/cognition/graph/`.
- LLM is called via the wrapper, not directly. Provider switching = env var change.
- Checkpoint corruption is recoverable by replaying from last good checkpoint.

## Implementation Pointer

- `app/cognition/graph/` — node + builder definitions
- `app/cognition/llm/client.py` — `CognitionLLMClient` provider wrapper
- `app/cognition/memory/checkpoint.py` — checkpoint id helpers
- `app/cognition/memory/store.py` — `MemoryStore` for cross-session memory
