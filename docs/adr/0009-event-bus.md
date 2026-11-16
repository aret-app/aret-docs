# ADR-0009: Event bus — NATS JetStream

- **Status:** Accepted
- **Date:** 2026-11-15
- **Deciders:** @balditommaso, @lorenzopieri99

## Context
Services exchange a small number of events (`user.created`, `user.deleted`, `track.created`…) for work like GDPR account deletion and track processing (spec §6.4). Events must not be lost when a service crashes. In Phase 3, live location sharing needs fast publish/subscribe to push updates to the app. Everything must run on small or free VMs.

## Options considered
1. **Apache Kafka** (KRaft mode) — the industry standard for event streaming with the largest ecosystem (Debezium, Kafka Connect). JVM broker needing ~1–2 GB of RAM, partitions and retention to tune; the Rust client `rdkafka` wraps a C library, which complicates static multi-arch images. Built for volumes far beyond ours.
2. **NATS JetStream** — mature CNCF project (since 2011), a single ~20 MB binary using tens of MB of RAM; durable streams with replay, key-value store, request/reply and low-latency pub/sub; official pure-Rust client `async-nats` on tokio.
3. **No broker for the MVP** (Postgres outbox table polled by the worker) — fewest moving parts, but a later migration once live features need pub/sub.

## Decision
Use **NATS JetStream**, with the **transactional outbox pattern** in each service: events are written to an `outbox` table in the same database transaction as the data change, and a relay publishes them to JetStream.

## Consequences
- Tiny footprint; fits next to the other containers on one VM.
- One system covers durable events now and live location pub/sub in Phase 3.
- Smaller ecosystem than Kafka: no Kafka Connect; change-data-capture, if ever needed, goes through Debezium Server's NATS sink or our own outbox relay.
- If event volume or integrations ever outgrow NATS, the outbox pattern keeps services independent of the broker, so switching (e.g. to Kafka) only touches the relay and consumers.
