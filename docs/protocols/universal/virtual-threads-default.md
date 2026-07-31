---
id: PP-20260730-vt01
title: "Virtual threads are the default concurrency model — Uni/reactive only for streaming and messaging"
type: principle
scope: universal
applies_to: "Any Quarkus/Vert.x-based module choosing a concurrency model"
severity: important
refs:
  - docs/adr/0005-virtual-threads-replace-reactive-tier.md
violation_hint: "New code uses Uni<T> for non-streaming work, or wraps blocking code in Uni.createFrom().item(). Use synchronous code on virtual threads instead."
created: 2026-07-30
---

## Default: virtual threads + synchronous code

All new code uses synchronous blocking style. Virtual threads make blocked threads cheap — the original motivation for reactive wrappers (avoid thread-per-request scalability limits) no longer applies.

Do NOT use `Uni<T>` or `Multi<T>` for:
- Database access (use blocking JPA/JDBC)
- REST endpoints (use @RunOnVirtualThread)
- Event handlers (use @RunOnVirtualThread + void return)
- SPI implementations (return T directly, not Uni<T>)
- Worker execution (synchronous on virtual thread)

## Exceptions: keep reactive for these patterns

| Pattern | Why |
|---------|-----|
| Reactive Messaging (Kafka, AMQP) | SmallRye Reactive Messaging API is Multi<T> by design |
| Multi<T> streaming with backpressure | No virtual thread equivalent for backpressure |
| Postgres LISTEN/NOTIFY broadcasters | Event-driven pub/sub — reactive PG client is correct |
| SSE endpoints | Already @RunOnVirtualThread |
| WebSocket endpoints | Already @RunOnVirtualThread |

## No dual-tier architecture

One bean per service. No Reactive*Service / Reactive*Store pairs. No @IfBuildProperty reactive gating. No BlockingReactiveParityTest. No build-time reactive config.

## Fan-out

Use StructuredTaskScope (Java 21+) instead of Uni.combine().all().unis(). For simple cases, sequential calls on virtual threads are sufficient.

## Error handling

Use try-catch instead of .onFailure().recoverWithUni().
