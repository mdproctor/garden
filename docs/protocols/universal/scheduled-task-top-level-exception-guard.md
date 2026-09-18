---
id: PP-20260918-17ff14
title: "Scheduled background tasks must have top-level exception guards"
type: rule
scope: universal
applies_to: "Any Runnable passed to ScheduledExecutorService.scheduleAtFixedRate or scheduleWithFixedDelay"
severity: critical
violation_hint: "scheduleAtFixedRate task body has try-catch on inner operations but not at the outermost level — an exception from setup code permanently kills the scheduler"
created: 2026-09-18
---

`ScheduledExecutorService.scheduleAtFixedRate` permanently and silently suppresses all future executions if the `Runnable` throws any uncaught exception. No log entry is produced. Wrap the entire task body in a top-level `try { ... } catch (Exception e) { LOG.log(SEVERE, "...", e); }` — not just individual sub-operations. Per-phase or per-step catches are not sufficient; setup code, capability checks, and iteration over tenants all sit outside those catches and can kill the scheduler.
