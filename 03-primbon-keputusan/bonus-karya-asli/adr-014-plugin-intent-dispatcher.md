---
aliases:
  - "Chatbot Kasir - ADR-014: Plugin Intent Dispatcher + Capabilities Markdown As Config"
---

# Chatbot Kasir - ADR-014: Plugin Intent Dispatcher + Capabilities Markdown As Config

**Status:** Accepted — ~2026-06-29 (era v2-routing), didokumentasikan retroaktif 2026-09-06

## Context

Routing V1 (graph-only) bikin nambah intent = ngedit core. Butuh cara nambah/matiin intent tanpa nyentuh pipeline, plus jalur bypass-LLM buat intent deterministik.

## Decision

- Intent executor = modul plugin `app/cognition/routing/<category>/<UPPERCASE_INTENT>.py`, auto-discovery via `pkgutil` (`dispatcher.py`).
- Intent aktif/nonaktif dikontrol dari section `## Status Intent` di `capabilities.md` — di-parse regex saat import, **fail-fast** kalau format mismatch.
- `validate_registry()` cross-check: tiap intent ACTIVE wajib punya modul Python + skill `.md`. Startup mati kalau gak konsisten.
- Teacher routing pakai confidence gate (`_TEACHER_MIN_CONFIDENCE = 0.70`, fallback map per-intent); dispatcher route 100% kalau confident, fallback ke graph kalau nggak.

## Rationale

Nambah intent = nambah 1 file py + 1 file md, nol edit core. Fail-fast di startup lebih murah daripada intent hilang senyap di runtime. Konfigurasi di markdown bikin non-koder (QA) bisa baca status intent.

## Alternatives Considered

Registry dict manual di Python (nolak: gampang lupa sync sama skill md); config YAML terpisah (nolak: nambah file sumber kebenaran ketiga).

## Forward Implication

`capabilities.md` = production config, bukan dokumentasi (Pitfall 29). Orang baru WAJIB dikasih tau sebelum nyentuh folder context.

## Implementation Pointer

`app/cognition/routing/dispatcher.py` (parse + validate), `app/cognition/routing/__init__.py` (confidence gates), `app/cognition/context/capabilities.md`.
