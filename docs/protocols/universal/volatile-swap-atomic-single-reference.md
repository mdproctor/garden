---
id: PP-20260918-57a402
title: "Volatile swap-and-reset must be atomic via single-reference pattern"
type: rule
scope: universal
applies_to: "Any @ApplicationScoped bean with two or more volatile fields that are swapped together in a reset/snapshot operation"
severity: important
violation_hint: "Two volatile fields swapped in separate assignments — concurrent readers see one new and one old, losing data in the gap"
created: 2026-09-18
---

When two related volatile fields must be swapped atomically (e.g. a counts map and a timestamps map in a swap-and-reset pattern), bundle them into a single volatile record reference. Two separate volatile writes create a window where a concurrent reader sees the new value of field A but the old value of field B, causing data loss or inconsistent snapshots. Example: `private volatile State state = new State();` where `record State(ConcurrentHashMap<...> counts, ConcurrentHashMap<...> timestamps) {}`.
