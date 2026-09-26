# Phase 11 — Route projection and ETA

Use this prompt only when Phase 11 is the current phase.

## Execution prompt

```text
You are implementing ONLY Phase 11 — Route projection and ETA of bug_gps.

Goal:
Convert raw live location into a defensible arrival estimate.

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
- product_spec.md
- architecture.md
- api_contract.md
- testing.md
- implementation_tracker.md

First inspect:
- git status
- current source tree
- relevant existing implementation
- previous phase status and exit gate

Do NOT begin if the previous phase is not verified. If the previous phase is broken, use docs/development/prompts/RECOVERY.md first.

Phase scope:
- route geometry handling
- nearest/appropriate route segment
- distance remaining
- smoothed speed
- ETA state machine
- stale/invalid handling
- confidence/unknown states

Explicitly forbidden in this phase:
- push notification
- machine-learning prediction
- paid routing APIs unless explicitly approved

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
- stationary bus
- moving bus
- known route geometry
- low speed
- impossible speed
- stale fix
- off-route case

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
ETA remains stable enough for a controlled demonstration and does not show a confident value when the input is invalid or stale.

STOP after Phase 11. Do not start Phase 12.

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
