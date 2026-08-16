---
aliases:
  - "Chatbot Kasir - ADR-008: Product Matcher Source-Containment Score"
---

# Chatbot Kasir - ADR-008: Product Matcher Source-Containment Score

Related: [Chatbot Kasir - Architecture Decisions](../../02-PROJECTS/chatbot-kasir/02-architecture-decisions.md), [Chatbot Kasir - Domain Glossary](../../02-PROJECTS/chatbot-kasir/03-domain-glossary.md), [Chatbot Kasir - Known Pitfalls](../../02-PROJECTS/chatbot-kasir/07-known-pitfalls.md)

## Status

Accepted — 2026-05-29 (commit `db17cc2`)

## Context

The product matcher in `app/cognition/operational/catalog.py:_build_candidates` originally used `_score_similarity` (SequenceMatcher ratio) and `_score_token_overlap` (Jaccard). Candidate admission threshold was 0.7. QA discovered that single-token queries like `"minyak"` were failing for `"Minyak Goreng"` (12 chars compact vs source 6 chars → ratio ≈ 0.667, below 0.7), while shorter siblings `"Minyak Tanah"` (11 chars compact) and `"Minyak Angin"` passed at 0.706.

Result: merchant could not see all 3 minyak products in an ambiguity prompt.

## Decision

Added a `_score_source_containment` metric: `len(intersect(source_tokens, target_tokens)) / len(source_tokens)`. Combined into candidate confidence via `max(...)`.

Semantically: "if every token of the user's query appears as a token in the product name, that product is a candidate, regardless of length."

## Rationale

- Captures the merchant's mental model: typing `"minyak"` means "I want anything starting with minyak".
- Length-invariant. Future product names like "Minyak Sayur Premium" (18 chars compact) still admit.
- Composes cleanly with existing scores via `max(...)`.

## Alternatives Considered

- **Lower the threshold to 0.65** — rejected. Brittle. A longer future product name would fail again.
- **Substring match** — rejected. Too many false positives ("min" matching "vitamin", "domain", etc.).

## Forward Implication

- Single-token queries should now admit all token-prefixed catalog items as candidates.
- Test verified: `"minyak"` against catalog with 3 Minyak products → ambiguity prompt with 3 candidates (QA pass 2 confirmed).

## Implementation Pointer

- `app/cognition/operational/catalog.py:_score_source_containment` — the new scoring function
- `app/cognition/operational/catalog.py:_build_candidates` — usage in `max(...)` confidence
- Tests in `tests/test_operational_catalog.py`
