---
id: PP-20260808-340f01
title: "CDI @Decorator for cross-cutting AgentProvider concerns — @Priority(APPLICATION + N) ordering"
type: rule
scope: platform
applies_to: "Any cross-cutting concern that wraps AgentProvider — rate limiting, circuit breaking, observability, auditing"
severity: important
refs:
  - docs/protocols/casehub/ai-agent-provider-cdi-priority.md
  - docs/protocols/casehub/alternative-extension-patterns.md
violation_hint: "Using @Alternative @Priority to wrap AgentProvider for a cross-cutting concern — @Alternative replaces, @Decorator wraps. Or adding rate limiting / observability directly into a provider implementation."
created: 2026-08-08
---

Cross-cutting behaviour on `AgentProvider` (rate limiting, circuit breaking, observability) uses CDI `@Decorator` — not `@Alternative`, not abstract base classes, not per-implementation duplication. `@Alternative` replaces an implementation (only one wins the priority contest); `@Decorator` wraps whatever implementation wins (the concern is orthogonal to the backend). The gate module (`agent-gate/`) establishes the precedent at `@Priority(Interceptor.Priority.APPLICATION)` (2000). Future decorators use `@Priority(APPLICATION + N)` where N determines decorator ordering — lower priority executes first (closer to the caller). A decorator at `APPLICATION - 10` runs before the gate; `APPLICATION + 10` runs after it (closer to the bean). This complements the existing three-pattern CDI ladder (`@DefaultBean` → `@ApplicationScoped` → `@Alternative @Priority`) documented in `alternative-extension-patterns.md` and `ai-agent-provider-cdi-priority.md`.
