---
id: PP-20260704-d0b9f3
title: "@TestTransaction tests calling REQUIRES_NEW helper beans must use unique entity names"
type: rule
scope: universal
applies_to: "Any @QuarkusTest using @TestTransaction where the service under test delegates to a @Transactional(REQUIRES_NEW) CDI helper bean"
severity: important
refs: []
violation_hint: "ConstraintViolationException on a unique index in a @TestTransaction test that worked before extracting a helper bean with REQUIRES_NEW — the helper's committed data from a prior test run collides with the new test's insert"
created: 2026-07-04
---

`@TestTransaction` wraps the test method in a transaction that rolls back after completion. When the service under test delegates to a helper bean annotated `@Transactional(TxType.REQUIRES_NEW)`, the helper's transaction commits independently — the test transaction's rollback does not undo it. Tests that relied on rollback for cleanup (using fixed entity names like `"auth-refactor"`) will collide across test classes sharing the same Quarkus context. Use unique names (e.g. `"auth-refactor-" + System.nanoTime()`) and `QuarkusTransaction.requiringNew()` instead of `@TestTransaction` when the service path includes REQUIRES_NEW delegation.
