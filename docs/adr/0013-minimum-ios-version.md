# ADR-0013: Minimum iOS version — iOS 18 / watchOS 11

- **Status:** Accepted
- **Date:** 2026-11-15
- **Deciders:** @balditommaso, @lorenzopieri99

## Context
The architecture spec proposed iOS 17. The prototype already targets **iOS 18 and watchOS 11** with Swift 6, and includes an Apple Watch app and a Live Activity.

## Options considered
1. **iOS 17** — matches the original spec; would require auditing the code for iOS 18-only APIs.
2. **iOS 18** — what the prototype uses. iOS 18 supports the same iPhones as iOS 17 (iPhone XS/XR and newer), so no devices are lost.
3. **iOS 26** — newest APIs, but excludes people who haven't updated yet.

## Decision
Minimum **iOS 18** for the iPhone app and **watchOS 11** for the Watch app.

## Consequences
- No code changes needed; `CLLocationUpdate.liveUpdates()`, `CLBackgroundActivitySession`, the Observation framework and Swift 6 concurrency are all available.
- Revisit once a year, after each new iOS release, based on adoption.
