# Independent Phase Review Prompt

Use this before moving from a completed phase to the next phase. This is deliberately review-oriented rather than implementation-oriented.

```text
Act as an independent reviewer, not the implementer.

Review only the current phase against docs/development/implementation_plan.md and its documented exit gate.

Read:
- .agent/Agents.md
- relevant product/architecture/engineering docs
- current phase section in docs/development/implementation_plan.md
- docs/development/implementation_tracker.md
- git diff for the phase

Verify:
- scope is complete
- forbidden future scope was not added
- relevant tests exist
- tests actually pass
- manual acceptance test is evidenced
- failure states are handled
- security requirements are respected
- code matches architecture
- docs match code
- no suspicious dead code or speculative abstractions were introduced

Do not rewrite the implementation during review unless explicitly asked.

Return:
PHASE REVIEW
- Scope: PASS/FAIL
- Tests: PASS/FAIL
- Manual acceptance: PASS/FAIL
- Security: PASS/FAIL
- Architecture compliance: PASS/FAIL
- Documentation sync: PASS/FAIL
- Exit gate: PASS/FAIL
- Blocking findings:
- Non-blocking findings:
```
