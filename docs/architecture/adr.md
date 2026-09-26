# Architecture Decision Records

This is the change-control document for architecture.

## ADR-001 — Use React Native + native Kotlin Android layer

**Status:** Accepted

**Decision:** Use React Native for UI/application presentation and Kotlin for the Android location foreground service.

**Reason:** The developer already knows JavaScript/TypeScript, while continuous background location is an Android-native concern that should remain under explicit native lifecycle control.

**Consequences:** Two languages and a native bridge must be maintained. This is intentional.

## ADR-002 — Use bare React Native rather than Expo Go

**Status:** Accepted

**Decision:** Use a bare React Native Android project.

**Reason:** The project deliberately owns a foreground service and native Kotlin module.

## ADR-003 — Use Fused Location Provider

**Status:** Accepted

**Decision:** Use Google Play services' Fused Location Provider on the primary Android build.

**Reason:** Appropriate Android API for practical location acquisition, while allowing the app architecture to hide the provider behind a location abstraction.

## ADR-004 — Use Android Foreground Service for driver tracking

**Status:** Accepted

**Decision:** Driver continuous tracking runs in an Android location foreground service.

**Reason:** The driver workflow requires location collection while the UI is backgrounded/locked, and Android provides a defined foreground-service path for long-running location work.

## ADR-005 — Use Supabase as MVP backend

**Status:** Accepted

**Decision:** Supabase provides Auth, PostgreSQL, RLS, Realtime, and server-side functions.

**Reason:** It minimizes infrastructure while retaining a strong relational data model and database-level authorization.

## ADR-006 — Use MapLibre React Native

**Status:** Accepted

**Decision:** MapLibre React Native is the map renderer.

**Reason:** Open-source/native map rendering and compatibility with the modern React Native architecture.

## ADR-007 — Do not store student home coordinates

**Status:** Accepted

**Decision:** Student config stores a selected stop, walking duration, and buffer; it does not store precise home location.

**Reason:** The feature does not require exact home coordinates and minimizing sensitive location collection is preferable.

## ADR-008 — Do not use a dedicated Node/Express server initially

**Status:** Accepted

**Decision:** Use Supabase APIs/Realtime/Edge Functions for MVP backend operations.

**Reason:** A separate API server adds infrastructure without providing a required MVP capability.

## ADR-009 — Use FCM for background push notifications

**Status:** Accepted

**Decision:** Firebase Cloud Messaging provides push delivery.

**Reason:** Standard Android push mechanism and appropriate for student notifications when the app is not actively open.

## Change request template

When proposing a new architecture decision:

```text
### ADR-NNN — <title>

Status: Proposed | Accepted | Rejected | Superseded

Context:

Decision:

Alternatives considered:

Reason:

Consequences:

Migration/testing impact:
```

No architectural replacement becomes “official” until it is recorded here.

## Dependency change record

Use this section for important upgrades.

```text
Date | Old | New | Reason | Tests | Approved
```

No dependency upgrade should be performed casually during a feature phase.
