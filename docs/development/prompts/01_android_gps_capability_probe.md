# Phase 1 — Android GPS capability probe

Use this prompt only when Phase 1 is the current phase.

## Execution prompt

```text
You are implementing ONLY Phase 1 — Android GPS capability probe of bug_gps.

Goal:
Prove the application can obtain a trustworthy one-shot location on a real Android device.

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
- implementation_tracker.md

First inspect:
- git status
- current source tree
- relevant existing implementation
- previous phase status and exit gate

Do NOT begin if the previous phase is not verified. If the previous phase is broken, use docs/development/prompts/RECOVERY.md first.

Phase scope:
- Kotlin location access
- minimum required runtime permissions
- one-shot current location
- display latitude, longitude and accuracy
- explicit handling of denied permission, disabled location and unavailable fix

Explicitly forbidden in this phase:
- foreground service
- background tracking
- Supabase
- MapLibre
- ETA
- offline queue

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
- permission granted/denied
- location services disabled
- current location fix
- invalid/no-fix state
- real-device accuracy reading

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
A real Android phone can obtain and display a valid coordinate and accuracy without any background tracking.

STOP after Phase 1. Do not start Phase 2.

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
