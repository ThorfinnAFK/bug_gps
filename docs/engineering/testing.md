# Testing Strategy

## 1. Test pyramid

```text
                 E2E / Real device
                /              \
         Integration          Manual GPS
          /       \
      Unit tests  Component tests
```

The GPS/foreground-service path requires real Android testing. Emulator-only validation is insufficient for final acceptance.

## 2. Unit tests — TypeScript

Test:

- ETA formula
- route projection helpers
- leave-home calculation
- payload normalization
- stale-location detection
- event deduplication
- validation schemas

Example test cases:

```text
12 min ETA - 5 min walking - 2 min buffer = 5
0 ETA never produces negative leave time
stale location => ETA unavailable
invalid coordinate => rejected
```

## 3. Component tests

Test:

- Driver Start button states
- tracking status rendering
- permission-denied UI
- Student bus marker rendering
- stale-data badge
- ETA card states
- loading/error/empty states

## 4. Native Kotlin tests

Test:

- location validation
- jump detection
- timestamp validation
- deduplication
- retry/backoff policy
- service command handling
- notification state creation

## 5. Android instrumentation tests

Verify where practical:

- permission flow
- service start while app visible
- notification creation
- service stop behavior
- lifecycle behavior

## 6. Database tests

Mandatory RLS scenarios:

### Student isolation

- student A can read authorized route
- student A cannot read student B's private preference
- student A cannot write locations
- student A cannot access unrelated school's trips

### Driver isolation

- driver A can write only to assigned active trip
- driver A cannot write to driver B's trip
- driver cannot write to completed trip

### Admin

- admin can perform intended management actions
- normal users cannot perform admin writes

## 7. Realtime tests

Verify:

- location insert triggers expected event
- unauthorized subscribers receive nothing useful
- reconnect resumes subscription
- duplicate event does not duplicate marker state

## 8. E2E test flows

### E2E-001 Driver start

```text
Launch
→ Login driver
→ Assigned bus visible
→ Start trip
→ tracking state ACTIVE
→ foreground notification visible
```

### E2E-002 Background tracking

```text
Start trip
→ lock phone
→ wait/drive
→ collect several location points
→ verify timestamps advance
```

### E2E-003 Student live view

```text
Login student
→ route loaded
→ active trip found
→ map opens
→ bus marker visible
→ new location causes marker update
```

### E2E-004 ETA

```text
Known simulated location
→ target stop known
→ ETA calculated
→ ETA changes as bus advances
```

### E2E-005 Network outage

```text
Start trip
→ disconnect network
→ continue movement
→ queue remains bounded
→ reconnect
→ pending points upload
```

### E2E-006 Permission denial

```text
Deny location
→ Start trip
→ no crash
→ clear explanation
→ no fake ACTIVE state
```

## 9. GPS test matrix

Test at least:

- stationary phone
- walking speed
- vehicle movement
- weak indoor signal
- network disconnected
- GPS disabled
- low battery mode where available
- phone locked
- app swiped from recents
- process restart after trip state persistence

## 10. Performance tests

Measure:

- map frame responsiveness under location events
- JS memory usage
- native service CPU use
- location update frequency
- network payload rate
- battery drain during a representative 30–60 minute trip

Record results in `build.md`.

## 11. Release gate

No release candidate if:

- TypeScript compile fails
- lint fails
- unit tests fail
- RLS tests fail
- native tests fail
- critical E2E flow fails
- driver background tracking has not been tested on real hardware
- release build cannot be installed
- crash reporting is not verified
