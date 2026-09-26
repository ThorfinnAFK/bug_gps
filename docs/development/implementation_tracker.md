# Implementation Tracker

> This file is intentionally a live checklist. The AI agent must update it after every completed phase and never mark an item complete without test evidence.

## Status legend

- `[ ]` not started
- `[-]` in progress
- `[x]` complete and verified
- `[!]` blocked

## Phase tracker

| Phase | Status | Automated tests | Manual test | Evidence/commit |
|---|---|---|---|---|
| 0 Toolchain | [ ] | | | |
| 1 GPS probe | [ ] | | | |
| 2 Foreground service | [ ] | | | |
| 3 Native module | [ ] | | | |
| 4 Offline queue | [ ] | | | |
| 5 Supabase/RLS | [ ] | | | |
| 6 Auth/roles | [ ] | | | |
| 7 Trip lifecycle | [ ] | | | |
| 8 Location upload | [ ] | | | |
| 9 Map UI | [ ] | | | |
| 10 Realtime | [ ] | | | |
| 11 ETA | [ ] | | | |
| 12 Leave-home | [ ] | | | |
| 13 Notifications | [ ] | | | |
| 14 Observability | [ ] | | | |
| 15 Release candidate | [ ] | | | |
| 16 Demo package | [ ] | | | |

## Current build health

```text
TypeScript:      UNKNOWN
ESLint:          UNKNOWN
Jest:            UNKNOWN
Kotlin tests:    UNKNOWN
Android build:   UNKNOWN
RLS tests:       UNKNOWN
E2E tests:       UNKNOWN
Real-device GPS: UNKNOWN
Realtime:        UNKNOWN
Release build:   UNKNOWN
```

## Current known issues

- None recorded yet.

## Current technical debt

- None recorded yet.

## Build evidence log

### 2026-09-26

- Documentation baseline created.
- Stack frozen at planning level.
- No implementation code has been validated yet.

## Rule for AI agents

Do not convert `UNKNOWN` into `PASS` based on code inspection alone. Verification requires command output, test output, or a documented manual test.
