# Recovery / Broken-State Prompt

Use this when a build fails, tests regress, the AI made an unintended change, or the project has drifted from the docs.

```text
Stop feature development.

Treat the repository as a broken engineering state that must be diagnosed before new work continues.

Read:
- .agent/Agents.md
- docs/architecture/architecture.md
- docs/engineering/techstack.md
- docs/engineering/testing.md
- docs/development/build.md
- docs/development/implementation_tracker.md
- docs/architecture/adr.md

Inspect:
- git status
- git diff
- recent build/test output
- dependency changes
- changed configuration

Determine whether the failure is:
1. code defect
2. configuration/toolchain issue
3. flaky/environment issue
4. incorrect test
5. documentation drift
6. architecture drift
7. accidental future-phase implementation

Do not hide the problem with:
- disabling tests
- suppressing compiler/linter errors
- broad exception swallowing
- weakening type checks
- changing unrelated architecture
- deleting functionality merely to make the build green

Fix the smallest root cause possible.

Then:
- rerun the failed check
- rerun the smallest relevant regression suite
- run the phase gate if appropriate
- sync the documentation using docs/development/prompts/SYNC.md

If the root cause requires an architecture or stack change, create/update an ADR before implementation.

Finish with:
RECOVERY RESULT
- Root cause:
- Fix:
- Tests rerun:
- Remaining issues:
- Architecture changed: YES/NO
- Phase can resume: YES/NO
```
