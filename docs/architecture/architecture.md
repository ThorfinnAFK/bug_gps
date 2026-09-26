# Architecture

## 1. System architecture

```text
                         SCHOOL BUS TRACKER

┌──────────────────────────── DRIVER DEVICE ────────────────────────────┐
│ React Native UI                                                        │
│   DriverDashboard                                                      │
│       │                                                                │
│       ▼                                                                │
│ Typed NativeLocation API                                               │
│       │ command/event boundary                                         │
│       ▼                                                                │
│ Kotlin LocationServiceModule                                            │
│       │                                                                │
│       ▼                                                                │
│ LocationForegroundService                                               │
│       │                                                                │
│       ├── permission/location-state checks                              │
│       ├── FusedLocationProviderClient                                  │
│       ├── location validator/filter                                    │
│       ├── local durable queue                                          │
│       └── uploader                                                      │
└──────────────────────────────┬────────────────────────────────────────┘
                               │ HTTPS / Supabase
                               ▼
┌────────────────────────────────────────────────────────────────────────┐
│                              SUPABASE                                  │
│                                                                        │
│ Auth ─ PostgreSQL ─ RLS ─ Realtime ─ Edge Functions                   │
│          │            │             │                                  │
│          │            │             └── notify/derived events          │
│          │            └──────────────── authorization                 │
│          └────────────────────────── state                             │
└──────────────────────────────┬─────────────────────────────────────────┘
                               │ Realtime / API
                               ▼
┌──────────────────────────── STUDENT DEVICE ────────────────────────────┐
│ React Native                                                          │
│   Session                                                               │
│   Route/Stop data                                                      │
│   Realtime subscription                                                 │
│         │                                                              │
│         ├── MapLibre → bus marker                                      │
│         ├── Turf → route position/distance                             │
│         └── ETA engine → leave-home recommendation                     │
└────────────────────────────────────────────────────────────────────────┘
```

## 2. Architectural boundaries

### Mobile JavaScript boundary

Owns:

- screens
- navigation
- user interactions
- session state
- rendering
- non-platform business logic
- realtime presentation
- ETA presentation

Must not own:

- long-running Android service lifecycle
- direct Android service startup internals
- privileged credentials
- low-level location listeners

### Native Android boundary

Owns:

- runtime location permission state
- foreground service lifecycle
- location callbacks
- service notification
- native persistence/queue needed for durable delivery
- native Android failure handling

Must expose only a small API to TypeScript.

### Backend boundary

Owns:

- authentication
- authorization
- canonical persistent state
- active trip state
- location ingestion rules
- cross-user data protection
- realtime publication
- server-side privileged operations

## 3. Proposed repository layout

```text
school-bus-tracker/
├── android/
├── src/
│   ├── app/
│   ├── navigation/
│   ├── screens/
│   │   ├── auth/
│   │   ├── student/
│   │   ├── driver/
│   │   └── admin/
│   ├── components/
│   ├── features/
│   │   ├── auth/
│   │   ├── trips/
│   │   ├── tracking/
│   │   ├── routes/
│   │   └── notifications/
│   ├── services/
│   │   ├── supabase/
│   │   ├── realtime/
│   │   └── native-location/
│   ├── domain/
│   │   ├── types/
│   │   ├── validation/
│   │   └── eta/
│   ├── hooks/
│   └── utils/
├── android/app/src/main/java/com/<org>/bustracker/
│   ├── MainActivity.kt
│   ├── MainApplication.kt
│   ├── bridge/
│   │   └── LocationServiceModule.kt
│   └── tracking/
│       ├── LocationForegroundService.kt
│       ├── LocationValidator.kt
│       ├── LocationRepository.kt
│       └── TrackingNotification.kt
├── supabase/
│   ├── migrations/
│   ├── seed.sql
│   └── functions/
├── tests/
├── e2e/
├── docs/
└── .github/workflows/
```

## 4. Driver trip lifecycle

```text
IDLE
 │
 │ user taps Start
 ▼
PREPARING
 │ permissions/location/network checks
 │ create/start trip
 ▼
TRACKING
 │
 ├── location callback → validate → persist → upload
 ├── temporary network loss → queue
 ├── permission disabled → degraded/error state
 └── service interruption → recovery attempt
 │
 │ user taps End
 ▼
STOPPING
 │ flush queue where possible
 │ mark trip ended
 ▼
COMPLETED
```

## 5. Tracking state model

The UI-visible tracking state should be explicit:

```text
IDLE
STARTING
ACTIVE
DEGRADED_NETWORK
DEGRADED_GPS
STOPPING
STOPPED
ERROR
```

Do not model this as a collection of independent booleans. State contradictions such as `isTracking=true` and `serviceRunning=false` must be represented and recoverable.

## 6. Native location pipeline

```text
FusedLocationProviderClient
        ↓
Location callback
        ↓
Normalize
        ↓
Validate timestamp
        ↓
Validate accuracy
        ↓
Detect impossible jump
        ↓
Optional speed/bearing sanity check
        ↓
Deduplicate / throttle
        ↓
Persist locally
        ↓
Upload
        ↓
Mark delivered
        ↓
Emit UI event if needed
```

## 7. Location rules

A location fix is considered usable only if:

- timestamp is not unreasonably old or in the future
- latitude/longitude are valid
- horizontal accuracy is below configured maximum
- movement from previous accepted location is plausible
- timestamp ordering is not broken

Suggested starting configuration:

```text
minimum update interval: 5 s
maximum reporting age: 20 s
maximum accepted accuracy: 100 m
stationary duplicate suppression: configurable
obvious-jump speed threshold: 150 km/h
upload batching: up to 1–5 points during transient outage
```

These are initial engineering defaults, not universal GPS truths. Measure on real devices and adjust with an ADR if needed.

## 8. Network pipeline

Normal case:

```text
Accepted location
  ↓
Local durable record
  ↓
Upload
  ↓
Server acknowledgement
  ↓
Mark delivered
```

Offline case:

```text
Accepted location
  ↓
Local durable record
  ↓
Upload fails
  ↓
Retry with exponential backoff + jitter
  ↓
Upload succeeds
  ↓
Mark delivered
```

The queue is bounded. Old queued points can be compacted if necessary; the latest location is more valuable than an unbounded historical queue.

## 9. Realtime student flow

```text
Student login
   ↓
Load assigned route/stop
   ↓
Resolve current active trip
   ↓
Subscribe to authorized location changes
   ↓
Receive latest point
   ↓
Update in-memory bus position
   ↓
Project onto route
   ↓
Compute route distance remaining
   ↓
Compute ETA
   ↓
Render marker + ETA
```

## 10. ETA engine

Inputs:

- accepted bus GPS position
- route geometry
- ordered stops
- historical/smoothed bus speed
- target stop

Algorithm for MVP:

1. Snap bus position to route geometry.
2. Determine route distance from bus position to target stop.
3. Maintain a smoothed speed estimate from recent accepted movement.
4. Clamp speed to a sane minimum/maximum for ETA calculation.
5. `ETA_minutes = route_distance / estimated_speed`.
6. Add a configurable uncertainty/safety buffer for the leave-home recommendation.

Do not claim that this is traffic-aware navigation.

## 11. Leave-home calculation

Inputs:

- target stop ETA
- student's walking duration in minutes
- configurable safety buffer

Formula:

```text
leave_in_minutes = max(0, bus_eta_minutes - walking_minutes - safety_buffer)
```

Example:

```text
bus ETA = 12 min
walking = 5 min
buffer  = 2 min

leave in = 5 min
```

## 12. Trip consistency rules

Backend must enforce:

- one active trip per bus
- one active trip per driver unless explicitly allowed
- trip route must match bus route
- location upload must reference an active trip
- driver must be authorized for that trip
- student subscription/read access must be limited to appropriate school/route data

## 13. Failure modes

### Driver loses network

Expected:

- continue GPS collection
- queue recent points locally
- show degraded state in driver UI/notification if useful
- retry upload
- do not terminate trip solely because internet disappears

### GPS disabled

Expected:

- location callbacks stop/degrade
- show actionable driver state
- do not silently claim tracking is healthy

### Poor accuracy

Expected:

- reject or mark degraded point
- keep last known good position visible with staleness indicator

### Service process killed

Expected:

- detect if practical
- persist desired tracking state
- recover only under Android-allowed lifecycle conditions
- never pretend the service is alive if it is not

### Student offline

Expected:

- display last known location with timestamp
- show stale badge
- remove/disable ETA when too stale

## 14. Security boundaries

The mobile app is an untrusted client.

Never trust:

- role strings sent by the client
- bus IDs chosen by a client
- trip IDs without authorization checks
- student stop IDs without ownership checks
- timestamps without validation
- reported speed without sanity checks

Authorization must be enforced in backend/database policies.

## 15. Why we are not using a separate Node/Express server initially

A dedicated Node backend would add API routing, WebSocket handling, deployment, secrets, and another operational surface. Supabase already provides the required primitives for the MVP.

A Node backend can be introduced later if the domain grows beyond what the current architecture handles. That would require an ADR.
