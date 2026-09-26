# Documentation and State Sync Prompt

Run this after every meaningful implementation block and ALWAYS before declaring a phase complete.

```text
Perform a repository synchronization pass after the implementation work just completed.

Read:
- .agent/Agents.md
- docs/architecture/architecture.md
- docs/architecture/database.md
- docs/architecture/api_contract.md
- docs/architecture/adr.md
- docs/engineering/techstack.md
- docs/engineering/security.md
- docs/engineering/testing.md
- docs/development/build.md
- docs/development/implementation_plan.md
- docs/development/implementation_tracker.md

Then inspect the actual changed files and git diff.

Check for:
- code/documentation contradictions
- undocumented dependencies
- accidental future-phase implementation
- changed API/data shapes
- changed database behavior
- changed permissions
- security regressions
- missing tests
- stale build instructions
- stale tracker status

Update documentation when the implementation has intentionally changed an agreed behavior.
If a design decision changed, update docs/architecture/adr.md before continuing.
If the frozen stack changed, update docs/engineering/techstack.md and record an ADR.
If a test command or build procedure changed, update docs/development/build.md or docs/development/Setup.md.

Update docs/development/implementation_tracker.md with:
- phase status
- tests actually executed
- manual test evidence
- relevant commit hash if available
- known issues
- technical debt

Append a dated entry to docs/development/build.md describing important verification performed.

Do not mark anything PASS or COMPLETE from static inspection alone.

Finish with exactly:
SYNC STATUS
- Code/documentation consistent: YES/NO
- Tests evidenced: YES/NO
- Security concerns: NONE / list
- Architecture drift: NONE / list
- Tracker updated: YES/NO
- Ready to continue current phase: YES/NO
```
