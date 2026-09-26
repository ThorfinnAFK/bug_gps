# Development Setup

## 1. Target environment

Primary development OS:

- Windows 11 with WSL2 Ubuntu 24.04 is supported for source tooling.
- Android Studio and the Android SDK should be installed on the host in the configuration recommended by the React Native/Android tooling.
- A physical Android phone is required for final GPS tests.

Linux/macOS native development also works if the environment satisfies the same Android/Node/JDK versions.

## 2. Required tools

Install:

- Git
- Node.js 22.x LTS, minimum 22.13.0
- npm
- JDK 17
- Android Studio stable
- Android SDK Platform 37
- Android SDK Build Tools 36.x as required by AGP 9.4.0
- Android Emulator (optional)
- ADB/platform-tools
- VS Code or another editor

AGP 9.4.0 currently lists JDK 17 and Gradle 9.6.0 compatibility.

Source: https://developer.android.com/build/releases/agp-9-4-0-release-notes

RN 0.87 currently requires Node >= 22.13.0 and raises compileSdk/build tools to 37.

Source: https://reactnative.dev/blog/2026/08/11/react-native-0.87

## 3. Android environment variables

Configure:

```text
ANDROID_HOME
JAVA_HOME
```

and ensure the following are on PATH as appropriate:

```text
platform-tools
cmdline-tools
emulator
```

Verify:

```bash
node --version
java -version
adb version
```

## 4. Android SDK packages

Install at minimum:


```text
Android SDK Platform 37
Android SDK Build-Tools 36.x
Android SDK Platform-Tools
Android SDK Command-line Tools
```

The exact installed build-tools patch may be selected by AGP/tooling; commit the project configuration actually used.

## 5. Create project

Use the official React Native Community CLI and pin the version after bootstrap.

Conceptually:

```bash
npx @react-native-community/cli@latest init SchoolBusTracker --version 0.87.x
```

Use the latest patch of the 0.87 line available at project start, then commit the resulting dependency lockfile.

Do not use Expo Go.

## 6. Android phone setup

On the test Android phone:

1. Enable Developer Options.
2. Enable USB debugging.
3. Connect to the development machine.
4. Accept the USB debugging trust prompt.
5. Verify:

```bash
adb devices
```

The device should appear as `device`.

## 7. Supabase setup

Create separate projects/environments for:

```text
Development
Staging (optional for mini-project)
Production/demo release
```

Create:

- project URL
- publishable client key
- database schema
- Auth configuration
- Realtime publication configuration
- Edge Functions if required
- FCM integration secrets if push notifications are enabled

Never place the service-role key in the mobile app.

## 8. Environment configuration

Use environment files that are ignored by Git:

```text
.env
.env.local
.env.development
.env.staging
.env.production
```

Commit only:

```text
.env.example
```

Example variables:

```text
SUPABASE_URL=
SUPABASE_PUBLISHABLE_KEY=
MAPTILER_KEY=
SENTRY_DSN=
```

Server-only variables belong in Supabase/CI secret stores, not the mobile environment.

## 9. Map setup

MapLibre React Native requires a map style/tile source. Use the approved provider from `techstack.md`.

Do not use the public OSM tile server as an uncontrolled production dependency.

Provide visible OpenStreetMap attribution when OSM-derived data is used.

## 10. Firebase / FCM setup

Only needed in the notification phase.

Configure the Android Firebase application for the package ID.

Keep server-side FCM credentials outside the mobile bundle.

Test notifications on at least two real devices.

## 11. Supabase migration workflow

Use the Supabase CLI for database migrations.

Expected flow:

```text
create migration
→ inspect SQL
→ apply locally/test
→ run RLS tests
→ commit migration
```

Never manually mutate production schema without a migration record.

## 12. First-run verification

After cloning:

```text
[x] Node version correct
[x] Java version correct
[x] adb works
[x] dependencies install
[x] TypeScript passes
[x] lint passes
[x] debug APK builds
[x] app launches on real device
```

Do not start feature development until Phase 0 passes.
