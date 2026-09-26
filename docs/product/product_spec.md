# Product Specification

## 1. Roles

### STUDENT

Can:

- log in
- view own profile
- view assigned bus/route
- view active trip for assigned route
- view current bus position
- view ordered stops
- choose a stop from their route
- configure walking duration
- view ETA
- view leave-home recommendation
- receive relevant notifications

Cannot:

- start/end trips
- publish bus locations
- access other schools' data
- select arbitrary bus IDs to follow

### DRIVER

Can:

- log in
- view assigned bus/route
- start assigned trip
- see tracking status
- stop assigned trip

Cannot:

- publish arbitrary coordinates for another driver/bus
- edit route definitions through the normal driver UI
- read unrelated students

### ADMIN

Can manage:

- users
- buses
- drivers
- routes
- stops
- trip assignments

Admin capabilities should be restricted in RLS or server functions.

## 2. Functional requirements

### AUTH-001

User can sign in with email/password.

Acceptance:

- valid credentials create session
- invalid credentials show user-safe error
- session survives app restart
- logout clears local session state

### DRIVER-001

Driver sees assigned bus and route.

### DRIVER-002

Driver can start a trip from a visible foreground screen.

Acceptance:

- location permission is granted first
- location services are enabled
- service starts successfully
- foreground notification appears
- state becomes ACTIVE

### DRIVER-003

Driver tracking persists through normal backgrounding/locking.

Acceptance:

- lock phone for >= 10 minutes during controlled drive/simulation
- location timestamps continue to advance
- service notification remains present

### DRIVER-004

Driver can end a trip.

Acceptance:

- location updates stop
- trip marked complete
- pending upload queue is flushed where possible
- UI returns to IDLE

### TRACK-001

The service validates incoming GPS points.

### TRACK-002

The service queues unsent location points during temporary network failure.

### TRACK-003

Duplicate and implausible points are suppressed or flagged.

### STUDENT-001

Student sees the active bus on a map.

### STUDENT-002

Student sees stop list in route order.

### STUDENT-003

Student selects a stop and walking duration.

### STUDENT-004

Student sees ETA.

### STUDENT-005

Student sees leave-home recommendation.

### STUDENT-006

Student sees stale-data status when live location is unavailable.

### NOTIFY-001

Student can receive relevant bus arrival/proximity notifications.

## 3. Non-functional requirements

### NFR-001 Reliability

Temporary network loss must not immediately terminate driver tracking.

### NFR-002 Security

All exposed database tables must use intentional RLS policies.

### NFR-003 Privacy

No precise student home coordinates in MVP.

### NFR-004 Battery

Tracking configuration should avoid unnecessarily high GPS frequency. Benchmark battery usage during a representative trip.

### NFR-005 Performance

Student map screen should remain responsive while realtime location events arrive.

### NFR-006 Observability

Release builds must report fatal crashes and unhandled errors through approved telemetry.

### NFR-007 Maintainability

Native tracking is isolated from UI logic through a small typed module boundary.

## 4. UX rules

- One primary action per screen.
- Driver Start/Stop controls must be impossible to trigger accidentally.
- Tracking status must be explicit.
- Students must see a timestamp/staleness indicator for bus position.
- ETA should show `Unavailable` rather than a misleading number when data is too stale.
- Never expose raw GPS payloads to normal users.

## 5. Demo data

Provide deterministic seed data for:

- one school
- two drivers
- two buses
- two routes
- 5–6 stops per route
- at least 3 students

The seed script must never contain production credentials.
