---
aliases:
  - "Chatbot Kasir - ADR-011: WIB (Asia/Jakarta) For All Merchant-Facing Date Filtering"
---

# Chatbot Kasir - ADR-011: WIB (Asia/Jakarta) For All Merchant-Facing Date Filtering

Related: [Chatbot Kasir - Architecture Decisions](../../02-PROJECTS/chatbot-kasir/02-architecture-decisions.md), [Chatbot Kasir - Known Pitfalls](../../02-PROJECTS/chatbot-kasir/07-known-pitfalls.md)

## Status

Accepted — 2026-05-28 (commit `6a84fb1`, fix/timezone-wib)

## Context

Any "today" / "hari ini" semantics in queries (daily report, transactions list) was previously filtered by UTC date. Merchants are in Indonesia. A sale at 22:00 WIB is at 15:00 UTC, but if it crosses midnight UTC, it ends up on a different UTC date than WIB date.

Result: merchant's "transactions hari ini" was wrong at edge hours.

## Decision

All merchant-facing date filtering uses Asia/Jakarta time (UTC+7), not UTC. Implementation via PostgreSQL `func.date(func.timezone('Asia/Jakarta', col))`.

## Rationale

- Merchants live in WIB. Their mental model of "today" is WIB-today.
- DST is not a concern (Indonesia does not observe DST). The offset is stable.
- Using DB-side timezone conversion keeps the query simple and consistent.

## Forward Implication

- Any new query that filters by "today" / specific date MUST use the WIB filter.
- Application code can use `today_wib()` from `app/shared/timezone.py`.
- For raw SQL, the pattern is `DATE(timezone('Asia/Jakarta', created_at)) = DATE(timezone('Asia/Jakarta', NOW()))`.
- The `WIB_PG_NAME` constant ensures the timezone name string is consistent.

## Implementation Pointer

- `app/shared/timezone.py` — exports `WIB_TZ`, `WIB_PG_NAME = "Asia/Jakarta"`, `today_wib()`
- `app/domain/operational/report.py` — daily report uses WIB filter
- `app/infrastructure/db/operational_repository.py:list_command_outbox_for_date` — uses WIB filter
- Tests in `tests/test_shared_timezone.py`
