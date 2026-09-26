# API and Data Contracts

## 1. Rule

Treat API/realtime payloads as contracts. TypeScript types, Kotlin data classes, validation schemas, and database columns must agree.

Use Zod on the JS side for untrusted payload validation.

## 2. Authentication

Supabase Auth manages sessions.

The mobile app stores the authenticated session using the approved secure persistence mechanism and refreshes it through the Supabase client.

Never manually construct JWTs on the client.

## 3. Logical operations

### Get profile

```text
GET /rest/v1/profiles?id=eq.<auth.uid>
```

In code, prefer the typed Supabase client rather than manually assembling URLs.

### Get active trip

Conceptually:

```text
trip where:
  status = ACTIVE
  route_id = user's authorized route
```

### Upload location

Payload:

```json
{
  "trip_id": "uuid",
  "recorded_at": "2026-09-26T18:30:00.000Z",
  "latitude": 10.5276,
  "longitude": 76.2144,
  "accuracy_m": 8.2,
  "speed_mps": 9.7,
  "bearing_deg": 83.0,
  "sequence_number": 1042,
  "device_event_id": "uuid-or-device-generated-id"
}
```

Validation:

- trip exists
- trip ACTIVE
- driver authorized
- timestamp plausible
- latitude [-90, 90]
- longitude [-180, 180]
- accuracy >= 0 if supplied
- speed >= 0 if supplied
- bearing [0, 360) if supplied
- payload size bounded

## 4. Realtime event

Canonical location event shape:

```ts
interface BusLocationEvent {
  id: string;
  tripId: string;
  recordedAt: string;
  receivedAt: string;
  latitude: number;
  longitude: number;
  accuracyM: number | null;
  speedMps: number | null;
  bearingDeg: number | null;
}
```

The application should normalize the database payload to this domain shape rather than leaking raw Supabase row naming everywhere.

## 5. Student preference

```ts
interface StudentRoutePreference {
  studentId: string;
  routeId: string;
  stopId: string;
  walkingMinutes: number;
  safetyBufferMinutes: number;
}
```

## 6. ETA domain model

```ts
interface EtaResult {
  available: boolean;
  etaMinutes: number | null;
  distanceMeters: number | null;
  estimatedSpeedMps: number | null;
  reason?:
    | 'NO_ACTIVE_TRIP'
    | 'NO_LOCATION'
    | 'STALE_LOCATION'
    | 'LOW_ACCURACY'
    | 'INVALID_ROUTE'
    | 'INSUFFICIENT_SPEED_DATA';
}
```

## 7. Error contract

Use stable error categories:

```text
AUTH_REQUIRED
FORBIDDEN
NOT_FOUND
VALIDATION_ERROR
TRIP_NOT_ACTIVE
LOCATION_UNAVAILABLE
GPS_PERMISSION_DENIED
LOCATION_SERVICES_DISABLED
NETWORK_UNAVAILABLE
RATE_LIMITED
SERVER_ERROR
UNKNOWN
```

Do not expose SQL errors or internal stack traces to users.

## 8. Idempotency / duplicates

The client should generate a `device_event_id` for each accepted point. The backend should make repeated uploads of the same event harmless.

If an idempotent database constraint is practical, add a unique index on the appropriate tuple, such as `(trip_id, device_event_id)`.

## 9. Timestamps

- Persist in UTC.
- Display in local device timezone.
- Never compare timestamps from different sources without normalization.
- Use device timestamp plus server received timestamp for diagnostics.

## 10. Versioning

MVP does not need public REST versioning, but domain contracts should carry a future-compatible version field if the system begins supporting multiple client versions.

Do not break a live database/client contract without a migration plan.
