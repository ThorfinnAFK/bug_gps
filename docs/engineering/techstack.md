# Frozen Technology Stack

**Freeze date:** 2026-09-26
**Policy:** Pin exact resolved dependency versions in the lockfiles/Gradle version catalog after project bootstrap. Do not use floating dependency versions in committed build files.

## 1. Stack overview

| Layer | Technology | Frozen baseline | What it does | Why this choice |
|---|---|---|---|---|
| Mobile runtime | React Native | 0.87.x | Cross-platform app shell/UI | User already knows JS/TS; current active RN line as of freeze date; supports modern New Architecture |
| UI language | TypeScript | version supplied by RN template, pinned | Application types and logic | Stronger contracts for location/trip/realtime payloads |
| JS engine | Hermes V1 | bundled/default with RN 0.87 | Executes JS | Default RN engine with current performance/tooling path |
| Android native | Kotlin | 2.2.x-compatible baseline | Android-specific services/APIs | Best fit for modern Android platform APIs and foreground services |
| Android build | AGP | 9.4.0 baseline | Build/package Android app | Current stable AGP release verified at freeze date and supports API 37 |
| Android build | Gradle | 9.6.0 baseline | Build orchestration | AGP 9.4 compatibility baseline |
| JDK | Java | 17 | Android build/runtime toolchain | AGP 9.4 compatibility baseline |
| Android SDK | compile/target | 37 | Compile and target Android | RN 0.87 raises compile SDK/build tools to 37 |
| Minimum Android | minSdk | 24 | Oldest supported device | MapLibre supports 23+, but 24 keeps device matrix and modern Android support simpler |
| Location | Google Play services Location | FusedLocationProviderClient | Location fixes | High-level fused provider suitable for Android tracking |
| Background tracking | Android Foreground Service | `location` FGS | Long-running driver tracking | Required architecture for persistent location tracking while app is not foreground UI |
| RN ↔ native | Turbo Native Module / New Architecture native module boundary | RN 0.87 New Architecture | Commands/events between TS and Kotlin | Keeps platform-specific tracking native while exposing a small typed API |
| Maps | MapLibre React Native | 11.x-compatible with RN 0.87 | Map UI, markers, route layers | Open source, native maps, compatible with modern RN architecture |
| Map data/style | Hosted vector tile/style provider | MapTiler initially | Map tiles/style | Avoid dependence on public OSMF tile servers for production traffic; provider is replaceable |
| Geospatial math | Turf.js | 7.x | Distance, route projection, geometry math | Avoid hand-rolling every geographic calculation |
| Backend platform | Supabase | current production platform; pin client SDK major 2.x | Auth, DB, Realtime, edge logic | One integrated backend with PostgreSQL and RLS |
| Backend client | `@supabase/supabase-js` | 2.x | Mobile API/auth/realtime client | Official JS client |
| Database | PostgreSQL | Supabase managed Postgres | Users, buses, routes, trips, locations | Relational model matches domain |
| Authorization | PostgreSQL RLS | mandatory | Row-level access control | Security boundary at database level |
| Realtime | Supabase Realtime / Postgres Changes | current GA path | Push DB changes to clients | Avoid polling for moving bus state |
| Push notifications | Firebase Cloud Messaging | current stable Android SDK | Background student notifications | Standard Android push delivery path |
| Local persistence | SQLite | Android-native/React Native supported | Offline queue and small local state | Durable local queue for short outages |
| Secure local secrets | Android Keystore + encrypted storage wrapper | platform | Tokens/secure state | Do not store secrets in plain text |
| Client state | React hooks + small typed store | no heavy state framework initially | UI/session state | Keep app simple until state complexity proves need |
| Navigation | React Navigation | current RN-compatible release | Screen navigation | Mature RN navigation solution |
| Validation | Zod | current stable 3/4-compatible API; pin exact version | Validate API/realtime payloads | Prevent malformed data from spreading through app |
| HTTP | fetch / platform networking | platform | Simple authenticated requests | No need for a second HTTP framework initially |
| Testing — JS | Jest + React Native Testing Library | RN-compatible versions | Unit/component tests | Fast feedback |
| Testing — native | JUnit + AndroidX Test | current compatible | Kotlin/unit/instrumentation tests | Verifies service/native behavior |
| Testing — E2E | Maestro | current stable | Black-box mobile acceptance tests | Simple readable flows for AI-maintained project |
| Static analysis — TS | TypeScript compiler | bundled/pinned | Type checking | Prevent contract drift |
| Static analysis — JS | ESLint | RN-compatible | Lint | Prevent common code problems |
| Formatting | Prettier | pinned | Formatting | Deterministic diffs |
| Static analysis — Kotlin | ktlint + optional detekt rules | pinned | Native code quality | Catch style and maintainability issues |
| CI | GitHub Actions | repository-native | Automated verification | Cheap, reproducible gates |
| Crash reporting | Sentry | current SDK compatible with RN/Android | Crash/error visibility | Production diagnostics |
| Source control | Git + GitHub | current | Versioning/review | Mandatory for AI-generated changes |

## 2. Architecture choice: bare React Native, not Expo Go

The application deliberately uses a **bare React Native project**.

Reason: the core requirement is a custom Android foreground service and native Kotlin location layer. We want direct ownership of the Android project and build configuration instead of hiding the core behavior behind a managed abstraction.

Expo can support custom native code through development builds, but that does not provide the learning/ownership benefit we want for this project.

## 3. Exact Android baseline

At bootstrap:

- Android compileSdk = 37
- Android targetSdk = 37
- Android minSdk = 24
- JDK = 17
- AGP = 9.4.0
- Gradle = 9.6.0
- Kotlin = compatible 2.2.x baseline
- Node.js >= 22.13.0; prefer current Node 22 LTS compatible patch
- React Native = 0.87.x latest patch available at bootstrap, then lock the exact resolved version

If the latest patch of 0.87.x changes after initial bootstrap, **do not auto-upgrade** during implementation. Upgrade only through an ADR and full regression test.

## 4. Why React Native 0.87

RN 0.87 is the active stable line as of the freeze date. The official release notes state that it requires Node >= 22.13.0, Kotlin 2.0+, supports AGP 9, and uses compileSdk/build tools 37. It also makes the Strict TypeScript API the default and continues the New Architecture path.

Source: https://reactnative.dev/blog/2026/08/11/react-native-0.87

## 5. Why Kotlin + foreground service

The driver phone must continue obtaining location while the UI is not actively visible. Android requires a declared location foreground-service type and appropriate permission prerequisites for this use case. The tracking service must therefore be native Android code.

Source: https://developer.android.com/develop/background-work/services/fgs/service-types

Source: https://developer.android.com/develop/background-work/services/fgs/restrictions-bg-start

## 6. Why MapLibre

MapLibre React Native wraps MapLibre Native for Android/iOS and currently requires React Native >= 0.80; its v11+ line requires the New Architecture. That matches RN 0.87's architecture direction.

Source: https://maplibre.org/maplibre-react-native/docs/setup/getting-started/

## 7. Why Supabase

Supabase gives this project PostgreSQL, authentication, realtime database changes, and RLS without forcing a separate API server for the MVP.

The mobile app uses only the public/publishable client key. Server-side privileged credentials must remain server-side.

Sources:
- https://supabase.com/docs/guides/auth
- https://supabase.com/docs/guides/realtime/postgres-changes
- https://supabase.com/docs/guides/database/postgres/row-level-security

## 8. Map provider policy

Do **not** make the public `tile.openstreetmap.org` service the production tile backend for a deployed application. OSMF states its public raster tile service is best-effort and subject to usage restrictions, caching, identification, and anti-bulk-download requirements. Use a hosted provider such as MapTiler or self-hosted tiles for production, and preserve required attribution/license information.

Source: https://operations.osmfoundation.org/policies/tiles/

## 9. No unnecessary technologies

Do not add any of the following without an ADR:

- Redux/MobX/Zustand if simple hooks/context are sufficient
- GraphQL
- WebSocket server outside Supabase Realtime
- Redis
- Kafka
- Kubernetes
- Microservices
- Firebase Database
- Firestore
- custom routing servers
- ML ETA model
- background student location collection

The project is intentionally designed as a modular monolith/mobile client, not a distributed-systems exercise.

## 10. Dependency policy

- Every package must have a reason.
- Remove unused packages.
- Prefer official or highly maintained packages.
- Pin exact versions in lockfiles.
- Never copy a dependency from Colota merely because Colota uses it.
- Run license checks before release.
- A dependency that requires architecture changes needs an ADR.
