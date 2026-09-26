# School Bus Live Tracker — Engineering Specification Pack

**Status:** Approved baseline / implementation-ready
**Date frozen:** 2026-09-26
**Target:** Production-grade engineering quality for a college mini-project; architecture is intentionally extensible to a real deployment.

## Purpose

This folder is the source of truth for building the School Bus Live Tracker in an AI-assisted IDE. Antigravity must read these documents before changing architecture, dependencies, database shape, security rules, or implementation order.

## Read order

1. `context.md` — product and problem definition
2. `product_spec.md` — functional and non-functional requirements
3. `techstack.md` — frozen technology decisions
4. `architecture.md` — system boundaries, flows, states, failure modes
5. `database.md` — schema and authorization model
6. `api_contract.md` — data contracts and backend interfaces
7. `security.md` — security/privacy rules
8. `implementation_plan.md` — isolated phases and quality gates
9. `Agents.md` — AI coding/agent operating rules
10. `Setup.md` — workstation and environment setup
11. `build.md` — build commandbook and build verification
12. `testing.md` — test strategy and acceptance gates
13. `deployment.md` — release and deployment process
14. `observability.md` — logging, crash reporting, health signals
15. `adr.md` — architecture decision record and change log
16. `implementation_tracker.md` — live project status

## Source-of-truth hierarchy

When two documents appear to conflict, use this order:

1. Explicit user-approved decision in `adr.md`
2. `techstack.md` for dependency/technology choices
3. `architecture.md` for system boundaries
4. `security.md` for security/privacy constraints
5. `database.md` and `api_contract.md` for data contracts
6. `implementation_plan.md` for phase sequencing
7. Other documents for supporting detail

Never resolve a conflict by silently changing the stack. Record the decision in `adr.md` first.

## Project headline

A driver phone runs a visible Android foreground location service during an active bus trip. GPS fixes are validated and uploaded to Supabase. Student phones subscribe to authorized live trip updates, render the bus on a map, and calculate an estimated arrival and a recommended time to leave home based on the student's selected stop and configured walking time.

## Hard principles

- Driver tracking must continue while the app is backgrounded/phone is locked, subject to Android OS behavior and user/device settings.
- Tracking starts from a deliberate foreground user action after permissions and location services are verified.
- Students do not need continuous location tracking.
- Never collect a student's precise home coordinates for the MVP.
- Do not put service-role keys or other server secrets in the mobile app.
- Every exposed Supabase table must have deliberate grants and Row Level Security policies.
- Every implementation phase must be runnable and testable independently before the next phase starts.
- AI-generated code is not trusted until tests and manual acceptance checks pass.
- No dependency or architecture changes without updating the relevant source-of-truth document and `adr.md`.

## Definition of done

The MVP is considered complete when a real Android driver device can start a trip, continue sending validated locations with the screen locked, and a second real Android device logged in as a student can see the bus move in near-real time, identify the student's configured stop, display an ETA, calculate a leave-home recommendation, and receive a notification. Security policies, failure recovery, tests, release signing, and documentation must also pass their gates.

## Current verified technology constraints

- React Native 0.87 is the active stable line as of 2026-09-26; it requires Node.js >= 22.13.0, supports Android Gradle Plugin 9, requires Kotlin 2.0+, and bumps Android compile SDK/build tools to 37. See official release notes.
- Android location foreground services require the `location` foreground-service type and `FOREGROUND_SERVICE_LOCATION` plus appropriate runtime location permission; the service must be started in a compliant app state.
- MapLibre React Native currently requires React Native >= 0.80 and its v11+ line requires the New Architecture.
- Supabase provides PostgreSQL, Auth, Realtime/Postgres Changes, and RLS; RLS must be deliberately configured for exposed data.

See the linked references at the bottom of the relevant technical documents.
