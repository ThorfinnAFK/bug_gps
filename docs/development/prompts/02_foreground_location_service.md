# Phase 2 — Foreground location service

Use this prompt only when Phase 2 is the current phase.

## Execution prompt

```text
You are implementing ONLY Phase 2 — Foreground location service of bug_gps.

Goal:
Implement continuous driver location tracking while the app is backgrounded/locked.

Before changing code, read:
- .agent/Agents.md
- docs/product/context.md
- docs/product/product_spec.md
- docs/architecture/architecture.md
- docs/engineering/techstack.md
- docs/engineering/security.md
- docs/engineering/testing.md
- docs/development/Setup.md
- docs/development/build.md
- docs/development/implementation_plan.md
- docs/development/implementation_tracker.md
- docs/development/prompts/SYNC.md
- docs/development/prompts/REVIEW.md

Also read these phase-specific documents:
- architecture.md
- security.md
- testing.md
- build.md
- implementation_tracker.md

First inspect:
- git status
- current source tree
- relevant existing implementation
- previous phase status and exit gate

Do NOT begin if the previous phase is not verified. If the previous phase is broken, use docs/development/prompts/RECOVERY.md first.

Phase scope:
- LocationForegroundService in Kotlin
- location foreground-service declaration
- persistent notification
- start/stop lifecycle
- continuous location callbacks
- explicit tracking state
- diagnostic logging safe for development

Explicitly forbidden in this phase:
- backend upload
- student UI
- MapLibre
- ETA
- authentication integration beyond what service startup requires

Implementation requirements:
1. Make the smallest coherent change that satisfies the phase.
2. Preserve existing architecture and frozen technologies.
3. Add or update tests with the feature.
4. Handle relevant success, denial, unavailable, stale, duplicate and failure states.
5. Do not add speculative abstractions.
6. Do not introduce a new dependency unless the dependency rule is satisfied and documentation is updated.
7. Do not change database migrations already applied.
8. Do not hard-code secrets or environment credentials.

Expected verification:
- start from visible activity
- lock phone for sustained period
- receive multiple location updates
- stop tracking
- missing permission
- location disabled
- service lifecycle edge cases

After implementation:
1. Run targeted automated tests.
2. Run typecheck/lint/build checks affected by the change.
3. Perform the manual acceptance test(s), on real Android hardware where applicable.
4. Record exact commands and outcomes in docs/development/build.md.
5. Run docs/development/prompts/SYNC.md.
6. Run docs/development/prompts/REVIEW.md as an independent check.
7. Update docs/development/implementation_tracker.md using evidence only.
8. If an intentional architecture or technology decision changed, create/update docs/architecture/adr.md before declaring completion.

Phase exit gate:
A real Android phone continues obtaining location while locked and clearly exposes active tracking through the foreground notification.

STOP after Phase 2. Do not start Phase 3.

Final response must contain:
PHASE RESULT
- Goal achieved:
- Files changed:
- Tests executed:
- Manual verification:
- Documentation updated:
- Known limitations:
- Exit gate: PASS/FAIL
- Commit:
```
