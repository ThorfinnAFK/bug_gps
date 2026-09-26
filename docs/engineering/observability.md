# Observability

## 1. Goals

We need to answer:

- Did the driver service start?
- Is location healthy?
- Are uploads succeeding?
- Is the student receiving realtime data?
- Is ETA unavailable because of a known reason?
- Did the application crash?

## 2. Mobile structured events

Use structured events, not arbitrary strings.

Examples:

```text
tracking.start_requested
tracking.permission_denied
tracking.service_started
tracking.service_stopped
location.accepted
location.rejected_accuracy
location.rejected_jump
location.queue_added
location.upload_success
location.upload_retry
location.upload_failed
realtime.connected
realtime.disconnected
eta.unavailable
```

Do not include raw latitude/longitude in general telemetry unless explicitly approved for short-lived debugging.

## 3. Key metrics

### Driver

- trip start success rate
- service start failure count
- accepted location rate
- rejected location rate by reason
- queue depth
- upload success rate
- upload latency
- current location age

### Student

- realtime connection state
- current location age
- ETA availability rate
- marker update interval

### Backend

- location insert rate
- RLS denial rate
- function errors
- database latency
- realtime disconnect/error rate

## 4. Sentry

Use Sentry for:

- fatal crashes
- unhandled exceptions
- meaningful native errors
- selected backend function errors

Avoid sending raw location streams as breadcrumbs/events.

## 5. Health definitions

A driver's tracking session is HEALTHY when:

```text
service running
AND
location age < threshold
AND
recent upload success
```

A student live view is HEALTHY when:

```text
subscription connected
AND
latest location age < threshold
```

## 6. Stale-location thresholds

Starting defaults:

```text
< 15s   live
15–60s  delayed
> 60s   stale
```

These are product/UI thresholds, not universal GPS truths. Adjust through observed testing and record the decision.

## 7. Privacy

Logs should never become an accidental second location database.

Prefer IDs, statuses, counters, durations, and reason codes over raw coordinates.
