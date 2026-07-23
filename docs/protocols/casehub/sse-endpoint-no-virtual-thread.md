# Protocol: SSE Endpoints Must Not Use @RunOnVirtualThread

**Applies to:** All casehubio repos — any JAX-RS endpoint that uses `SseEventSink`,
`@Produces(SERVER_SENT_EVENTS)`, or returns `Multi<T>` for streaming.

---

## Rule

**Never annotate an SSE streaming endpoint with `@RunOnVirtualThread`.**

SSE endpoints using `SseEventSink` + `void` return are long-lived streaming
connections. `@RunOnVirtualThread` is designed for short-lived request-response
handlers where the virtual thread runs the method body and is then released.

The interaction between `@RunOnVirtualThread` and SSE `SseEventSink` is
undocumented in Quarkus as of 2026-07. There is no official guidance confirming
that the virtual thread is released when the `stream()` method returns while the
`SseEventSink` stays open. Combining them risks:
- Virtual thread monopolisation for the lifetime of the SSE connection
- Undocumented dispatch behaviour that may change between Quarkus versions
- Memory pressure from unbounded virtual thread creation (one per SSE client)

---

## What SSE Endpoints Should Do Instead

### 1. Keep SSE endpoints on the event loop

SSE endpoints with `SseEventSink` + `void` return run on the event loop in
RESTEasy Reactive. Leave them there — the method body registers emitters and
returns quickly.

### 2. Offload blocking calls to virtual threads explicitly

If the SSE setup method needs to make a blocking call (e.g., fetching initial
unread count from a JPA store), offload it explicitly:

```java
private static final ExecutorService VIRTUAL_EXECUTOR =
    Executors.newVirtualThreadPerTaskExecutor();

@GET
@Produces(MediaType.SERVER_SENT_EVENTS)
public void stream(@Context SseEventSink eventSink, @Context Sse sse) {
    // Non-blocking setup: capture principal, register emitter
    String userId = principal.actorId();
    String tenancyId = principal.tenancyId();
    connections.computeIfAbsent(key, k -> /* register emitter */);

    // Offload blocking store call to virtual thread
    VIRTUAL_EXECUTOR.execute(() -> {
        try {
            long count = store.unreadCount(userId, tenancyId);
            sendUnreadCount(eventSink, sse, count);
        } catch (Exception e) {
            LOG.errorf(e, "Failed to fetch initial unread count for user %s", userId);
        }
    });
}
```

### 3. CDI @ObservesAsync handlers can block directly

`@ObservesAsync` CDI event handlers run on the managed executor, not the event
loop. Blocking store calls are safe there — no offloading needed:

```java
void onNotificationCreated(@ObservesAsync NotificationCreated event) {
    // Runs on managed executor thread — blocking is fine
    try {
        long count = store.unreadCount(userId, tenancyId);
        sendUnreadCountToUser(userId, tenancyId, count);
    } catch (Exception e) {
        LOG.errorf(e, "Failed to fetch unread count for user %s", userId);
    }
}
```

---

## Where @RunOnVirtualThread IS Correct

| Endpoint type | @RunOnVirtualThread | Why |
|---------------|--------------------|----|
| Regular REST (`GET`, `POST`, etc.) returning `T` | Yes | Short-lived, virtual thread released on return |
| `@ConsumeEvent` handler returning `void` | Yes | Short-lived event processing |
| SSE with `SseEventSink` + `void` return | **No** | Long-lived connection, undocumented interaction |
| SSE returning `Multi<T>` | **No** | Already reactive, runs on event loop |
| WebSocket endpoints | **No** | Long-lived connection, same risk as SSE |

---

## Qhorus Specific

Qhorus postgres-broadcaster modules (`LISTEN/NOTIFY`) are SSE/WebSocket by nature.
These stay reactive — `@RunOnVirtualThread` does not apply. Same for any channel
I/O that is genuinely async.

---

## References

- [Quarkus Virtual Threads Guide](https://quarkus.io/guides/virtual-threads)
- [Quarkus REST Virtual Threads Guide](https://quarkus.io/guides/rest-virtual-threads)
- [Virtual Thread Monopolisation Discussion](https://github.com/quarkusio/quarkus/discussions/36011)
- [Security + @RunOnVirtualThread Discussion](https://github.com/quarkusio/quarkus/discussions/48263)
- casehubio/parent#384 — discovered during reactive tier retirement

---

## Violation Hint

An SSE endpoint annotated with `@RunOnVirtualThread` or a class-level
`@RunOnVirtualThread` on a resource that contains SSE methods.
