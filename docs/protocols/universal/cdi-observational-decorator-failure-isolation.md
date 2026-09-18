---
id: PP-20260918-e7a320
title: "CDI observational decorators must isolate side-effect failures"
type: rule
scope: universal
applies_to: "Any CDI @Decorator that fires events, records tracking, or writes to secondary stores after the delegate operation succeeds"
severity: critical
violation_hint: "event.fire() or secondary store.store() called without try-catch after delegate succeeds — observer exception masks successful operation"
created: 2026-09-18
---

When a CDI decorator's purpose is observational (tracking, notification, affect recording), its side-effect code must be wrapped in try-catch so that failures never propagate to the caller. The delegate operation already succeeded — an observer throwing undoes that success from the caller's perspective. Use a `safeFire(Event<T>, T)` utility or equivalent try-catch-log wrapper. Contrast with the delegate call itself, which should propagate failures normally.
