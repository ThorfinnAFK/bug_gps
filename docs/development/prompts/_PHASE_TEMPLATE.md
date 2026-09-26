# Phase Prompt Template

Use this structure for phase-specific prompts.

```text
You are implementing ONE isolated phase of bug_gps.

Current phase: <PHASE NUMBER — NAME>

Required reading:
- .agent/Agents.md
- docs/product/context.md
- docs/product/product_spec.md
- docs/architecture/architecture.md
- docs/engineering/techstack.md
- docs/engineering/security.md
- docs/engineering/testing.md
- docs/development/implementation_plan.md
- docs/development/implementation_tracker.md
- <phase-specific docs>

Before coding:
1. inspect repository state
2. confirm previous phase exit gate passed
3. inspect relevant existing code
4. state the exact files/components you expect to change

Implement ONLY this phase.

Do NOT implement any future phase unless explicitly required by this phase.
Do not introduce dependencies without following the dependency rule in Agents.md.
Do not silently alter architecture.

For implementation:
- make the smallest coherent change
- keep boundaries typed
- add tests with the feature
- handle normal, denied, unavailable, stale, duplicate, and failure states relevant to this phase

Verification:
- run targeted tests first
- run build/lint/type checks affected by the change
- perform the manual acceptance test on real Android hardware when required
- record exact commands/results

Before completion:
- run docs/development/prompts/SYNC.md
- run docs/development/prompts/REVIEW.md
- update implementation_tracker.md only with verified evidence
- update build.md with important verification evidence
- create an ADR if an approved architectural decision changed
- commit only if repository workflow requires it

STOP after this phase. Do not start the next phase automatically.

Final report:
PHASE RESULT
- Implemented:
- Files changed:
- Tests executed:
- Manual verification:
- Known limitations:
- Documentation updated:
- Commit:
- Exit gate: PASS/FAIL
```
