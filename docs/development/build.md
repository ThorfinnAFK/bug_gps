# Build and Verification Log

## 1. Purpose

This file is the project's operational build diary and commandbook. It answers:

- How do we install dependencies?
- How do we build debug/release?
- What checks must pass?
- What was last verified?

## 2. Expected commands

> Exact package-manager commands may follow the generated RN template. Once the project exists, copy the final commands here and freeze them.

### Install

```bash
npm ci
```

If the project uses another package manager, update the repository decision explicitly and commit the lockfile.

### Type check

```bash
npm run typecheck
```

### Lint

```bash
npm run lint
```

### Unit/component tests

```bash
npm test -- --runInBand
```

### Android debug build

```bash
cd android
./gradlew assembleDebug
```

### Android clean build

```bash
cd android
./gradlew clean assembleDebug
```

### Release build

```bash
cd android
./gradlew bundleRelease
```

The release keystore must be supplied through secure CI/local configuration, never committed.

## 3. Device install

Use a real Android device whenever testing GPS.

```bash
adb devices
```

Debug install example:

```bash
adb install -r android/app/build/outputs/apk/debug/app-debug.apk
```

## 4. Log inspection

Useful Android commands:

```bash
adb logcat
adb shell dumpsys location
adb shell dumpsys activity services
adb shell dumpsys notification
```

Filter application logs by the chosen app tag once logging is implemented.

## 5. Required build gates

Before every phase completion:

```text
[x] TypeScript passes
[x] lint passes
[x] relevant unit tests pass
[x] Android debug build passes
[x] phase manual test passes
```

For phases touching RLS/database:

```text
[x] migrations pass
[x] RLS tests pass
```

For release candidate:

```text
[x] clean checkout builds
[x] signed release builds
[x] release installs
[x] driver background tracking passes on real phone
[x] student realtime view passes on second phone
```

## 6. Environment matrix

| Environment | Backend | Credentials | Purpose |
|---|---|---|---|
| local/dev | dev Supabase project | local env | development |
| CI/test | isolated test DB/project | CI secrets | automated tests |
| staging | staging backend | secure env | release validation |
| production | production backend | secure env | real deployment |

Never point a debug build casually at production data.

## 7. Build failure protocol

When a build fails:

1. Capture the first meaningful error.
2. Determine whether it is toolchain, dependency, code, environment, or generated-file drift.
3. Fix the smallest root cause.
4. Re-run the failed command.
5. Run the broader verification suite if the fix touches shared configuration.
6. Record the cause in this file if it is non-obvious.

Do not repeatedly regenerate the entire Android project as a blind fix.

## 8. Build log

### 2026-09-26

- No implementation build performed yet.
- Documentation pack created.

Future entries:

```text
YYYY-MM-DD — phase N
Command:
Result:
Device:
Notes:
Commit:
```
