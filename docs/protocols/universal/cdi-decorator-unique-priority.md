---
id: PP-20260918-5089a9
title: "CDI decorator priorities must be unique per SPI and documented"
type: rule
scope: universal
applies_to: "Any CDI @Decorator chain where multiple decorators intercept the same SPI interface"
severity: important
violation_hint: "Two @Decorator classes on the same interface share the same @Priority value — CDI ordering is undefined"
created: 2026-09-18
---

Every CDI decorator on the same SPI must have a unique `@Priority` value. Equal priorities produce undefined ordering across CDI implementations — behaviour may differ between Quarkus, Weld, and OWB, or between runs. Document the full decorator chain ordering in a comment or design doc so future decorators are inserted at the correct position. Lower priority = outermost (called first, sees final results on return).
