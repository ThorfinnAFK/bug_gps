# Security and Privacy

## 1. Threat model

Assume:

- any mobile client can be modified
- a user can inspect the APK/bundle
- client-side validation can be bypassed
- a malicious authenticated user can call APIs directly
- network requests can fail, replay, or arrive out of order
- a stolen session may be used until revoked/expired

Security must therefore be enforced server-side.

## 2. Secrets

Never commit:

- Supabase service-role key
- FCM server credentials
- Map provider secret/private keys
- signing keystore/password
- Sentry auth token
- CI tokens
- database passwords

Mobile app may contain only credentials intended for public/client use, such as the Supabase publishable key and a restricted map public key.

## 3. Supabase security

Required:

- RLS enabled on every exposed table.
- least-privilege grants
- no broad anonymous write policies
- service-role key only in server-side functions/CI secrets
- policies tested with student and driver identities

Source: https://supabase.com/docs/guides/database/postgres/row-level-security

## 4. Driver authorization

A location submission must prove:

```text
authenticated user
       ↓
role = DRIVER
       ↓
driver assigned to bus/trip
       ↓
trip active
       ↓
trip belongs to the user's school
       ↓
location accepted
```

Do not accept `driver_id` supplied by the client as authority.

## 5. Student authorization

A student may only receive information for their authorized school/route/trip.

Never create a public `get_all_bus_locations` endpoint.

## 6. Location privacy

Student precise home location is not collected in the MVP.

Driver location is operational data and should be retained only as long as required by product requirements.

Do not log raw GPS payloads into general application logs unless explicitly needed for a short-lived debugging session.

## 7. App permissions

Request only permissions needed for the active feature.

Driver:

- fine/coarse location as required by product
- foreground service location permission for supported Android versions
- notifications where required by Android version

Do not request background location merely because it sounds useful. Starting/using a location FGS has specific Android rules; permission order and service-start context matter.

Sources:
- https://developer.android.com/about/versions/14/changes/fgs-types-required
- https://developer.android.com/develop/background-work/services/fgs/restrictions-bg-start

## 8. Start-service security/lifecycle rule

The normal driver flow must be:

```text
visible driver screen
  ↓
request/check location permission
  ↓
check location services
  ↓
start location foreground service
  ↓
start location updates
```

Do not attempt to start a location FGS opportunistically from background JS.

## 9. Local storage

Sensitive local state must use secure storage appropriate for Android. The database queue itself must not contain reusable authentication secrets.

## 10. Replay protection

Use:

- device-generated event ID
- recorded_at
- received_at
- server-side sanity checks
- optional monotonic sequence number

Reject or ignore obvious duplicates and impossible timestamps.

## 11. Abuse prevention

Protect against:

- high-frequency location spam
- giant request payloads
- unauthorized trip selection
- repeated trip creation
- notification abuse

Add reasonable server-side rate limits and constraints.

## 12. Error handling

Users should see actionable messages such as:

```text
Location permission is required to start tracking.
```

not:

```text
java.lang.SecurityException: Permission denial...
```

Internal error details go to logs/telemetry.

## 13. Dependency and supply-chain security

- Keep lockfiles committed.
- Review dependency updates.
- Run package vulnerability scanning in CI.
- Avoid abandoned native modules for the core tracking path.
- Prefer official Android/React Native APIs where practical.

## 14. Release security checklist

- [ ] no secrets in Git history
- [ ] release keystore not committed
- [ ] RLS tests pass
- [ ] service-role key unavailable to mobile bundle
- [ ] API payloads validated
- [ ] rate limits tested
- [ ] debug logging disabled in release
- [ ] crash reporting enabled without sensitive GPS payload logging
- [ ] map attribution/license requirements satisfied
- [ ] privacy/data-retention policy reviewed for any real-school deployment
