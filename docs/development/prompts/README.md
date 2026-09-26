# Antigravity Prompt Pack

This directory contains the operational prompts for implementing `bug_gps`.

## How to use

1. Run `00_STARTUP.md` at the beginning of a session.
2. Identify the current phase from `docs/development/implementation_tracker.md`.
3. Run exactly that phase prompt.
4. Do not start the next phase in the same autonomous request.
5. Run `SYNC.md`.
6. Run `REVIEW.md`.
7. Only after the exit gate passes, advance the tracker and use the next phase prompt.

## Supporting prompts

- `SYNC.md` — synchronize code, docs, tests, tracker, and ADRs.
- `REVIEW.md` — independent acceptance review.
- `RECOVERY.md` — diagnose and repair broken states.
- `_PHASE_TEMPLATE.md` — template for adding a future phase.

## Authority

`docs/development/implementation_plan.md` remains the canonical phase definition. The prompts operationalize it; they do not override it.
