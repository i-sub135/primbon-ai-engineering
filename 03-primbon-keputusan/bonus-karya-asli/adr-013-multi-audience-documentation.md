---
aliases:
  - "Chatbot Kasir - ADR-013: Documentation Lives In Multiple Audience-Aware Files"
---

# Chatbot Kasir - ADR-013: Documentation Lives In Multiple Audience-Aware Files

Related: [Chatbot Kasir - Architecture Decisions](../../02-PROJECTS/chatbot-kasir/02-architecture-decisions.md), [Chatbot Kasir - Operational Rules](../../02-PROJECTS/chatbot-kasir/06-operational-rules.md), [Chatbot Kasir - Project Identity](../../02-PROJECTS/chatbot-kasir/01-project-identity.md)

## Status

Accepted — 2026-05-30 (formalized from QA documentation work 2026-05-29/30)

## Context

The project has multiple stakeholder audiences (engineering, project owner / PM, future engineers, AI agents). Documentation that serves one audience well usually serves another poorly. Writing everything for "everyone" produces docs that no one reads end-to-end.

## Decision

Documentation is split by audience:

- `docs/task.md` + `docs/task-child.md` + `docs/summary.md` — engineering source of truth (in source repo)
- `docs/qa-playbook.md` — stable QA methodology (in source repo)
- `docs/qa-report.md` — latest dated QA pass, engineering audience (in source repo)
- `docs/qa-report-mgmt.md` — same findings reframed for management (in source repo, local only, NOT committed)
- `docs/progress-report-mgmt.md` — overall progress for management (NOT committed)
- `docs/lampiran-timeline-mgmt.md` — sprint timeline appendix (NOT committed)
- `docs/V2/*.md` — domain specs (in source repo)
- Knowledge pack — agent handoff bundle in Knowledge-OS at `02-PROJECTS/chatbot-kasir/`
- `README.md`, `AGENTS.md` — root entry points in source repo

## Rationale

- A PM does not need to read about SequenceMatcher ratios.
- An engineer does not need to read business-risk framing for the third time.
- Audience-aware writing reduces "I have to ignore X" friction for every reader.
- AI agents specifically benefit from a separate handoff pack (Knowledge-OS) that does not pollute the source repo.

## Forward Implication

- Mgmt docs (`*-mgmt.md`) are intentionally NOT committed by default. They are local working artifacts.
- The QA report file gets replaced wholesale on each run; methodology is in the playbook. History stays inspectable via `git log -- docs/qa-report.md`.
- The knowledge pack (Knowledge-OS subfolder) is portable across AI tools and persists across source repo lifecycle.
- Adding new docs requires answering: who is the audience, and where does it belong?

## Implementation Pointer

- Source repo: `docs/` folder with audience-tagged filenames
- Knowledge-OS: `02-PROJECTS/chatbot-kasir/` for full pack, `10-AI-CONTEXT/chatbot-kasir-ai-handoff.md` for slim companion, `04-DECISIONS/chatbot-kasir/` for stable ADRs (this folder)
- Convention: `*-mgmt.md` = not committed; mgmt PDF outputs go to `~/Documents/`
