# ADR-0008: Authentication — self-hosted Ory Kratos

- **Status:** Accepted
- **Date:** 2026-11-15
- **Deciders:** @balditommaso, @lorenzopieri99

## Context
Accounts are needed for sync and sharing (the app works without one). We need email login, Sign in with Apple, later passkeys, email verification and recovery, secure sessions, and GDPR account deletion — at €0 hosting cost. Login code is where small teams most often introduce security bugs (hashing, token rotation, recovery flows, user enumeration, rate limiting).

## Options considered
1. **Own Rust `identity-svc`** (Argon2id, EdDSA JWT, rotating refresh tokens) — one tech stack and full control, but we write and maintain all security-sensitive code ourselves.
2. **Own service, passwordless** — no passwords to leak, but still our own security code, and an email sender from day one.
3. **Self-hosted Ory Kratos** — open source (Apache-2.0), widely used and reviewed, light Go service; registration, login, Apple, passkeys, TOTP, verification, recovery and native-app API flows already built.
4. **Self-hosted Zitadel** — full identity platform (AGPL) with admin UI; heavier to run on a free VM.

## Decision
Use **self-hosted Ory Kratos** for identities and sessions. It runs as a container with its own `kratos` database on our Postgres server. A thin Rust **`identity-svc`** stays as an adapter: it turns Kratos webhooks into our events (`user.created`) and runs account deletion (Kratos admin API + `user.deleted` event for the other services).

Our services authenticate requests with short-lived **JWTs produced by Kratos's session tokenizer**, verified locally with Kratos's public keys (JWKS). If the deployed Kratos version doesn't support this, the fallback is calling `/sessions/whoami` with a ~60 s cache in Valkey.

## Consequences
- We don't write password hashing, recovery or session code; we configure and operate Kratos instead (config, upgrades, security advisories, backups).
- Kratos's courier needs an SMTP provider (free tier) for verification and recovery emails.
- Emails are stored by Kratos, protected by encrypted volumes and encrypted backups; the spec's own email encryption with a blind index is dropped.
- The iOS app integrates Kratos's native-app flows (login, registration, Apple, later passkeys) in SwiftUI.
- Hosted Ory Network is not used; self-hosting keeps it free and keeps user data on hosting we choose (see ADR-0012).
