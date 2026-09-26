# Context

## 1. Problem

School bus users often have a simple information problem: the bus is physically moving, but students and parents do not know where it is or when they should leave home for their stop.

The project is inspired by the basic real-time tracking pattern used by ride-hailing systems, simplified for a school-bus domain.

## 2. Product concept

**School Bus Live Tracker** provides:

- Driver: starts and ends a bus trip from an Android phone.
- Driver phone: continuously obtains location while the trip is active.
- Backend: validates/stores the latest bus location and publishes updates.
- Student: sees the active bus on a map.
- Student: sees the next relevant stop and estimated arrival time.
- Student: sees a recommendation such as `Leave home in 5 min`, based on bus ETA, walking duration, and a safety buffer.

## 3. Primary users

### Driver

The driver needs a very low-interaction workflow because the phone is being carried in a moving vehicle.

Primary actions:

- View assigned bus/route.
- Start trip.
- Confirm tracking is active.
- End trip.

The driver should not need to interact with the map while driving.

### Student

The student needs passive information, not navigation.

Primary actions:

- Sign in.
- View assigned route/stop.
- See active bus location.
- See ETA.
- See recommended leave time.
- Receive arrival/proximity notification.

### Admin

Admin exists mainly to configure the system for the demo and future expansion.

Primary actions:

- Manage users.
- Assign drivers to buses.
- Assign buses to routes.
- Configure stops and stop order.
- Create/activate trips if needed for support.
- Inspect operational status.

## 4. Core user story

> As a student, I want to see my school bus's current location and an understandable estimate of when it will reach my stop, so I can decide when to leave home without repeatedly calling the driver.

## 5. Key value proposition

The product should answer two questions immediately:

1. **Where is my bus?**
2. **When should I leave?**

The moving marker is the mechanism. The ETA and leave-home recommendation are the user-facing value.

## 6. MVP scope

### In scope

- Android app
- React Native UI
- Kotlin native location service
- Foreground location service
- Driver and student roles
- Supabase Auth
- PostgreSQL schema
- RLS authorization
- Realtime bus location
- Map rendering
- Fixed route geometry
- Ordered stops
- GPS validation/filtering
- ETA based on route position and smoothed speed
- Leave-home recommendation
- Push/local notifications for important trip events
- Offline queue for short network interruptions
- Crash/error reporting
- CI checks
- Signed release APK/AAB

### Explicitly out of scope for the first release

- Ride matching
- Payments
- Driver navigation turn-by-turn
- Student continuous GPS tracking
- Student exact home coordinates
- Automatic school timetable integration
- Traffic-aware commercial routing API
- AI/ML ETA prediction
- Parent social features
- In-app chat
- Large-scale fleet optimization
- iOS
- Offline map downloads
- Multi-school SaaS administration beyond what is needed for the demo

## 7. Product assumptions

- Each bus has one active driver for a trip.
- Each trip is associated with exactly one bus and one route.
- A route has ordered stops with latitude/longitude.
- The driver phone has GPS/location service available.
- The driver's device can access the internet most of the time, but temporary outages must not lose the whole trip.
- Students can choose/configure their stop and walking duration.

## 8. Important privacy decision

Do not store a student's exact home latitude/longitude in the MVP.

Instead store:

- selected stop
- walking duration in minutes
- optional coarse preferences that do not identify a precise residence

This is enough to calculate a leave-home recommendation without collecting unnecessary sensitive location data.

## 9. Demo scenario

Use a route with 4–6 stops.

Example:

```text
Depot
  ↓
Stop A
  ↓
Stop B
  ↓
Stop C   ← student
  ↓
Stop D
  ↓
School
```

Test with:

- Device A = driver
- Device B = student
- Simulated route or real drive in a controlled environment

## 10. Success metrics for the mini-project

### Functional

- Driver can start/stop a trip.
- Valid location updates reach backend.
- Student receives location updates.
- Marker moves without refreshing the screen.
- ETA changes over time.
- Leave-home time changes with ETA.
- App behaves correctly when network temporarily disappears.

### Reliability

- No duplicate active trips for the same bus.
- Stale locations are detected and surfaced.
- GPS jumps are filtered.
- Service state survives normal app backgrounding.
- No crash when permission is denied.

### Security

- No service-role secret in mobile bundle.
- RLS prevents cross-school/cross-user unauthorized reads/writes.
- Driver can only publish for an assigned active trip.
- Student can only read trips allowed for their route/school.

## 11. Engineering philosophy

This is a mini-project, but it should be built as a **small production system**, not as a large prototype.

That means:

- simple core architecture
- strict boundaries
- measurable failure modes
- tests at every phase
- minimal permissions
- replaceable external services
- documented decisions
- no AI-generated black-box subsystems
