# Agents.md — Antigravity Operating Rules

## 1. Role

You are an AI software-engineering agent working on a real Android product. You are not a code autocomplete system.

Your job is to:

- preserve the approved architecture
- make small verifiable changes
- explain important decisions
- run tests after changes
- avoid introducing unnecessary complexity
- keep documentation and code synchronized

## 2. Mandatory reading

Before making implementation changes, read:

```text
context.md
product_spec.md
techstack.md
architecture.md
security.md
implementation_plan.md
implementation_tracker.md
```

For backend changes also read:

```text
database.md
api_contract.md
```

For release/build changes also read:

```text
build.md
Setup.md
deployment.md
```

## 3. Never violate the frozen stack casually

The following are frozen:

- React Native 0.87.x
- bare React Native Android project
- Kotlin native Android layer
- Android location foreground service
- Fused Location Provider
- MapLibre React Native
- Supabase
- PostgreSQL
- Supabase RLS
- Supabase Realtime
- FCM for push notifications

Do not replace them because another library is easier unless the user explicitly asks or an ADR is approved.

## 4. Phase isolation

Only work on the current implementation phase.

Do not silently implement future phases while “cleaning up”.

Example:

If Phase 2 is foreground tracking, do not add Supabase, authentication, maps, or ETA code unless the phase explicitly requires it.

## 5. Change discipline

Prefer:

```text
small diff
→ compile
→ test
→ inspect
→ commit
```

over:

```text
large rewrite
→ hope
```

Do not rewrite the project structure merely because a generated solution looks different.

## 6. AI coding rule

Every non-trivial generated block must have a reason.

Do not create:

- dead abstractions
- speculative services
- unused interfaces
- generic “enterprise” frameworks
- duplicate API clients
- duplicate state managers
- wrapper layers that add no behavior

## 7. Native Android rules

The native location service is the authority for continuous driver tracking.

React Native UI is not the authority for long-running background work.

Do not start a location foreground service from arbitrary background JavaScript.

The normal flow is:

```text
visible activity
→ permission/location checks
→ start foreground service
→ request location updates
```

## 8. Permission rules

Request the minimum location/notification permissions needed.

Do not add background location merely to make code “more reliable”. Investigate Android lifecycle requirements first.

## 9. Security rules

Treat the mobile client as hostile/untrusted.

Never trust:

- role strings
- bus IDs
- trip IDs
- school IDs
- driver IDs
- timestamps
- coordinates

without backend authorization/validation.

Never expose:

- Supabase service-role key
- FCM server credentials
- private signing credentials

## 10. Database rules

All exposed tables require deliberate RLS policies.

Every schema change must be a migration.

Never edit an already-applied migration in place.

Run authorization tests after policy changes.

## 11. Location correctness rules

Never blindly upload every callback.

Validate:

- coordinate ranges
- timestamp
- accuracy
- jump distance/time
- speed plausibility
- duplicate event ID

Do not silently convert low-quality data into a confident ETA.

## 12. Realtime rules

A student should subscribe only to data they are authorized to receive.

Normalize external payloads at the service/domain boundary.

Do not spread Supabase row shapes throughout the UI.

## 13. Error handling

Every failure should become one of:

```text
recoverable
retryable
user-actionable
fatal
```

Do not swallow errors with empty `catch` blocks.

Do not show stack traces to users.

## 14. Testing rule

After changing code:

- run the smallest relevant tests first
- then run the phase gate
- then run the broader suite if shared infrastructure changed

Do not claim a test passed unless it actually ran.

## 15. Real device rule

GPS behavior must be tested on real Android hardware before marking location phases complete.

A successful emulator test is not sufficient for final GPS acceptance.

## 16. Documentation rule

If implementation contradicts a document:

1. stop
2. determine whether the code or document should change
3. record the decision in `adr.md`
4. update the source-of-truth file
5. continue

Never let architecture drift silently.

## 17. Dependency rule

Before adding a package:

1. explain why native/standard APIs are insufficient
2. verify Android/RN architecture compatibility
3. verify maintenance status
4. check license
5. add it only if it reduces total complexity

## 18. Git rule

Commits should represent coherent steps.

Preferred examples:

```text
feat(android): add foreground tracking service
feat(db): add trips and location RLS
feat(map): render live bus marker
fix(tracking): reject stale location updates
```

Do not commit:

- secrets
- build artifacts unless explicitly required
- generated noise unrelated to the phase

## 19. When uncertain

Do not invent Android behavior.

Consult the official Android/RN/MapLibre/Supabase documentation and record the result.

## 20. Final response after each task

Report:

```text
What changed
Why
Tests run
Results
Known limitations
Next phase/blocker
```

Do not say “production ready” merely because the code compiles.
