# Deployment and Release

## 1. Environments

Use separate configurations for:

```text
DEV
STAGING
PRODUCTION/DEMO
```

Do not ship a debug build pointing at a production database.

## 2. Android build types

### Debug

- local development
- verbose diagnostics
- no production secrets
- test backend

### Release

- minification only after verifying native-service compatibility
- production/demo backend
- crash reporting enabled
- debug logs restricted
- signed artifact

## 3. Signing

Use an Android release keystore stored outside the repository.

Store passwords/paths in:

- local secure environment
- CI secret store

Never commit the keystore or password.

## 4. Versioning

Use semantic application versions where practical:

```text
MAJOR.MINOR.PATCH
```

Increment Android `versionCode` monotonically for each released build.

## 5. CI release gates

CI should run:

```text
install
→ typecheck
→ lint
→ unit tests
→ Kotlin tests
→ database/RLS tests
→ Android debug build
```

Release workflow additionally runs:

```text
clean checkout
→ release build
→ artifact inspection
→ optional signed AAB generation
```

## 6. Deployment of Supabase

Database:

- apply migrations
- run smoke/RLS tests

Edge Functions:

- deploy only from CI/approved workstation
- inject secrets through Supabase secrets

## 7. Pre-release manual checklist

### Driver

- [ ] login
- [ ] assigned bus visible
- [ ] permissions work
- [ ] start trip
- [ ] notification present
- [ ] lock phone
- [ ] move/drive
- [ ] locations continue
- [ ] stop trip

### Student

- [ ] login
- [ ] route loads
- [ ] stop selection works
- [ ] bus marker updates
- [ ] ETA works
- [ ] leave-home calculation works
- [ ] notification works

### Failure modes

- [ ] no network
- [ ] GPS disabled
- [ ] permission denied
- [ ] backend unavailable
- [ ] stale location
- [ ] app restart

## 8. Rollback

A release is rollback-safe only when:

- prior mobile build is available
- database migration is backward compatible or rollback plan exists
- server functions are versioned

Never assume mobile rollback can safely follow a breaking database migration.

## 9. Distribution

For academic work, an APK is sufficient for controlled installation.

For Play distribution, generate a signed AAB and complete Google Play requirements separately.

## 10. Production deployment warning

A real school deployment introduces privacy, child-safety, institutional approval, legal/data-protection, incident-response, and operational-support requirements beyond this student project specification.
