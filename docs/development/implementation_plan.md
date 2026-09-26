# Implementation Plan

## Rules for every phase

Each phase is **isolated and independently testable**.

A phase may not depend on an unfinished feature from a later phase.

For every phase:

1. Read the relevant specification files.
2. Implement only the phase scope.
3. Run automated tests.
4. Run the manual acceptance test.
5. Record evidence in `build.md`.
6. Update `implementation_tracker.md`.
7. Commit the phase before starting the next phase.

## Phase 0 — Repository and toolchain bootstrap

### Goal

Create the React Native Android project and reproducible development environment.

### Scope

- RN 0.87.x
- TypeScript
- Android SDK 37
- AGP 9.4.0
- Gradle 9.6.0
- JDK 17
- Node >= 22.13
- lint/format/test scripts
- Git hooks optional
- CI skeleton

### Must NOT include

- GPS
- Supabase
- maps
- auth

### Tests

- `node --version`
- package manager install succeeds
- TypeScript compile succeeds
- lint succeeds
- Android debug APK builds
- app launches on real Android device

### Exit gate

Blank app installs and launches from a clean checkout.

---

## Phase 1 — Android GPS capability probe

### Goal

Prove the device can obtain current location.

### Scope

- native Kotlin location access
- permission request
- one-shot current location
- display lat/lon/accuracy

### Must NOT include

- background service
- backend
- map

### Tests

- permission granted
- permission denied
- location services off
- GPS fix received
- invalid/no-fix state handled

### Exit gate

Real Android phone shows valid coordinates and accuracy after permission grant.

---

## Phase 2 — Foreground location service

### Goal

Make continuous driver tracking survive normal backgrounding/locking.

### Scope

- `LocationForegroundService`
- notification
- start/stop commands
- location callback loop
- explicit state machine
- service logs/diagnostics

### Tests

- start from visible screen
- lock screen >= 10 min
- multiple location updates received
- stop works
- permission missing does not create fake active state

### Exit gate

Real phone can track continuously while locked and exposes a persistent tracking notification.

---

## Phase 3 — Native module boundary

### Goal

Expose a small typed API to React Native.

### Scope

Commands:

```text
startTracking()
stopTracking()
getTrackingState()
```

Events:

```text
location
trackingStateChanged
trackingError
```

### Tests

- TS command reaches Kotlin
- Kotlin events reach JS
- app restart does not crash
- listener cleanup works

### Exit gate

React Native UI can control and observe native tracking without knowing Kotlin internals.

---

## Phase 4 — Local persistence and offline queue

### Goal

Protect location events against temporary network loss.

### Scope

- durable local location queue
- delivery state
- retry/backoff
- bounded queue

### Tests

- insert
- restart process
- network outage
- reconnect
- successful flush
- duplicate suppression
- queue bound

### Exit gate

Temporary outage does not lose all location points or stop tracking.

---

## Phase 5 — Supabase schema + migrations + RLS

### Goal

Build secure backend data foundation.

### Scope

- schema migrations
- seed data
- roles
- trips
- routes/stops
- locations
- RLS policies
- RLS tests

### Tests

- migrations cleanly apply
- seed succeeds
- student isolation
- driver isolation
- unauthorized location write blocked

### Exit gate

Database can be recreated from migrations and security tests pass.

---

## Phase 6 — Authentication and role routing

### Goal

Users can securely access the correct application experience.

### Scope

- Supabase Auth
- session persistence
- role-based navigation
- sign out

### Tests

- valid login
- invalid login
- expired session handling
- student/driver route selection
- logout

### Exit gate

Two seeded users can sign in and cannot enter the wrong role flow.

---

## Phase 7 — Driver trip lifecycle backend

### Goal

Connect driver start/stop to secure trip records.

### Scope

- start trip operation
- stop trip operation
- active-trip constraints
- assigned-bus enforcement

### Tests

- valid driver starts assigned trip
- second active trip blocked
- unauthorized driver blocked
- completed trip rejects location updates

### Exit gate

Backend owns canonical trip state.

---

## Phase 8 — Driver location upload

### Goal

Move validated driver locations from phone to backend.

### Scope

- location payload contract
- uploader
- idempotency
- queue integration
- staleness timestamps

### Tests

- normal upload
- duplicate upload
- outage/retry
- invalid payload blocked
- active-trip authorization

### Exit gate

Driver can publish a live trip location stream securely.

---

## Phase 9 — Map UI

### Goal

Visualize route, stops, and bus location.

### Scope

- MapLibre
- route layer
- stop markers
- bus marker
- attribution
- camera behavior

### Tests

- map loads
- route renders
- markers render
- location update moves marker
- stale location indicated

### Exit gate

Two-device demo can visually show the moving bus.

---

## Phase 10 — Realtime student tracking

### Goal

Remove polling and provide live updates.

### Scope

- authorized subscription
- reconnect handling
- event normalization
- marker state updates

### Tests

- update arrives without refresh
- unauthorized data not received
- reconnect restores stream
- duplicate event harmless

### Exit gate

Driver moves → student marker changes in near real time.

---

## Phase 11 — Route projection + ETA

### Goal

Turn raw location into meaningful arrival information.

### Scope

- route projection
- distance remaining
- smoothed speed
- ETA states
- invalid/stale handling

### Tests

- known geometry
- stationary bus
- movement
- stale location
- low speed
- impossible speed

### Exit gate

ETA is stable enough for a controlled demo and never displays obviously fake values.

---

## Phase 12 — Leave-home recommendation

### Goal

Answer when the student should leave.

### Scope

- stop selection
- walking duration
- safety buffer
- leave-in calculation

### Tests

- normal values
- negative result clamps to zero
- missing ETA
- stop not on route

### Exit gate

Student receives a clear recommendation linked to their configured stop.

---

## Phase 13 — Notifications

### Goal

Surface important state without forcing the app open.

### Scope

- driver tracking notification
- student arrival/proximity notification
- FCM registration
- server-side trigger path

### Tests

- foreground
- background
- locked screen
- duplicate notification suppression
- opt-out/permission states

### Exit gate

Notification flows work on real devices.

---

## Phase 14 — Observability and hardening

### Goal

Make failures diagnosable.

### Scope

- Sentry
- structured native logs
- staleness diagnostics
- backend health checks
- safe error reporting

### Tests

- deliberate test crash captured in non-production environment
- location failure captured without leaking raw GPS payloads
- user-facing error remains safe

### Exit gate

A failed driver trip can be diagnosed from telemetry plus local diagnostics.

---

## Phase 15 — Production build and release candidate

### Goal

Produce a signed, repeatable Android release artifact.

### Scope

- release build
- signing
- versioning
- environment separation
- privacy/config checks
- APK/AAB verification

### Tests

- clean release build
- install/update
- login
- background tracking
- realtime student view
- ETA
- notifications
- crash reporting

### Exit gate

All release tests pass on at least two real Android devices.

---

## Phase 16 — Final demonstration package

### Goal

Prepare the project for academic demonstration.

### Scope

- seeded demo data
- demo accounts
- architecture diagram
- test evidence
- screenshots
- build artifact
- README

### Exit gate

A reviewer can reproduce the core demo from the documentation without needing the original developer present.
