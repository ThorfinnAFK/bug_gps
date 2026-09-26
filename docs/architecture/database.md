# Database Design

## 1. Database principles

- PostgreSQL via Supabase.
- UUID primary keys for application entities.
- UTC timestamps (`timestamptz`).
- Foreign keys for relationship integrity.
- Soft deletion only where operationally useful; do not blindly add `deleted_at` everywhere.
- RLS enabled on every client-exposed table.
- Realtime enabled only on tables that genuinely need it.
- Service-role access stays server-side.

## 2. Core schema

### schools

```sql
id uuid primary key
name text not null
created_at timestamptz not null default now()
```

### profiles

```sql
id uuid primary key references auth.users(id) on delete cascade
school_id uuid not null references schools(id)
full_name text not null
role text not null check (role in ('STUDENT', 'DRIVER', 'ADMIN'))
created_at timestamptz not null default now()
updated_at timestamptz not null default now()
```

### buses

```sql
id uuid primary key
school_id uuid not null references schools(id)
bus_number text not null
route_id uuid null references routes(id)
is_active boolean not null default true
created_at timestamptz not null default now()
```

### routes

```sql
id uuid primary key
school_id uuid not null references schools(id)
name text not null
geometry jsonb not null
is_active boolean not null default true
created_at timestamptz not null default now()
```

For MVP, `geometry` can store a GeoJSON LineString. If geometry volume becomes significant, move to a PostGIS geometry column after an ADR. Do not introduce PostGIS only because it sounds production-grade.

### stops

```sql
id uuid primary key
route_id uuid not null references routes(id) on delete cascade
name text not null
latitude double precision not null
longitude double precision not null
sequence integer not null check (sequence > 0)
created_at timestamptz not null default now()
unique(route_id, sequence)
```

### bus_assignments

Associates drivers/buses to operational assignments.

```sql
id uuid primary key
bus_id uuid not null references buses(id)
driver_id uuid not null references profiles(id)
start_at timestamptz not null
end_at timestamptz null
```

### trips

```sql
id uuid primary key
bus_id uuid not null references buses(id)
route_id uuid not null references routes(id)
driver_id uuid not null references profiles(id)
status text not null check (status in ('ACTIVE', 'COMPLETED', 'CANCELLED'))
started_at timestamptz not null default now()
ended_at timestamptz null
```

The database must enforce or transactionally guarantee that only one active trip exists per bus.

### student_route_preferences

```sql
id uuid primary key
student_id uuid not null references profiles(id) on delete cascade
route_id uuid not null references routes(id)
stop_id uuid not null references stops(id)
walking_minutes integer not null check (walking_minutes between 0 and 180)
safety_buffer_minutes integer not null default 2 check (safety_buffer_minutes between 0 and 30)
updated_at timestamptz not null default now()
unique(student_id)
```

Do not store a home latitude/longitude.

### location_updates

```sql
id uuid primary key
trip_id uuid not null references trips(id) on delete cascade
recorded_at timestamptz not null
received_at timestamptz not null default now()
latitude double precision not null
longitude double precision not null
accuracy_m double precision null
speed_mps double precision null
bearing_deg double precision null
sequence_number bigint null
device_event_id text null
```

Index:

```sql
create index on location_updates (trip_id, recorded_at desc);
```

Only the latest/current location should need high-frequency reads. Historical trip points are for audit/debugging/demo history.

## 3. Recommended current-location strategy

For a mini-project, use `location_updates` as the authoritative append-only event stream and derive the current point with:

```sql
order by recorded_at desc limit 1
```

If realtime/write load grows, add a `trip_live_state` table containing exactly one row per active trip. That is an optimization, not required for the first implementation.

## 4. Authorization model

### Student

May read:

- own profile
- route assigned to own preference
- stops belonging to that route
- active trip for that route
- locations for that active trip
- own preference

May update:

- own stop/walking preference

May not:

- write location_updates
- modify trips
- modify bus assignments
- read arbitrary students

### Driver

May read:

- own profile
- assigned bus
- assigned route/stops
- own active trip

May write:

- start/end trip through approved operation
- location update for an authorized active trip

May not:

- publish to another driver's trip
- update route definitions

### Admin

Administrative writes should be implemented carefully, ideally through explicit RPC/Edge Function operations for complex state transitions.

## 5. RLS requirements

Enable RLS on every exposed table.

Examples of logical policies:

```text
student SELECT profile:
  auth.uid() = id

student UPDATE own preference:
  auth.uid() = student_id

student SELECT route:
  route is referenced by the user's authorized preference

student SELECT location:
  location.trip_id belongs to an active trip for the user's assigned route

location INSERT:
  authenticated driver + driver owns trip + trip active + school match
```

Do not implement policies by trusting client-supplied `role`, `school_id`, or `driver_id` without deriving/validating them from trusted database relationships.

Supabase's current RLS guidance explicitly recommends enabling RLS on exposed tables and managing grants and policies together.

Source: https://supabase.com/docs/guides/database/postgres/row-level-security

## 6. Realtime

Enable Realtime/Postgres Changes only for the minimal required live-location table(s).

Student subscriptions must be filtered to the authorized active trip where possible.

Source: https://supabase.com/docs/guides/realtime/postgres-changes

## 7. Data retention

Initial recommendation:

- Active live location: retained while trip active.
- Historical location updates: retain 7–30 days for demo/debugging, configurable.
- Student preference: retained while account exists.
- Logs/telemetry: follow service retention policy.

A real school deployment requires a privacy/data-retention review appropriate to the institution and jurisdiction.

## 8. Migration rules

- Never modify an applied migration in place.
- Create a new migration for every schema change.
- Seed data lives separately from production migrations.
- CI must run migrations against a disposable/test database before release.
- RLS tests must run in CI.
