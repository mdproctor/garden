---
id: PP-20260609-fe1300
title: "Callers of ChannelService.create() / ReactiveChannelService.create() must call gateway.initChannel() on the created channel"
type: rule
scope: platform
applies_to: "Any code that calls ChannelService.create() or ReactiveChannelService.create() and expects ChannelBackend implementations to receive messages on the created channel"
severity: critical
refs:
  - docs/protocols/casehub/gateway-backend-registration-ordering.md
garden_ref: "GE-20260609-9ee2ad"
violation_hint: "A newly created channel receives COMMAND messages but OpenClawChannelBackend (or any ChannelBackend) never handles them — ChannelGateway.fanOut() finds no registered backends for the channel because ChannelInitialisedEvent was never fired for it"
created: 2026-06-09
---

`ChannelService.create()` calls `channelGateway.initChannel()` internally (via `ChannelCreateHelper.createInNewTransaction()`). `ReactiveChannelService.create()` also calls `initChannel()` internally within its `withTransaction` block (offloaded to the worker pool). Callers of either `ChannelManager.create()` or `ReactiveChannelManager.create()` do NOT need to call `initChannel()` separately — it is handled by the create method. `findOrCreate()` paths (both binding-based and name-based) also go through the same creation logic, so backends are registered automatically for all channel creation paths. The startup hook (`ChannelGateway.onStart()`) continues to fire `initChannel` with `recovered=true` for all persisted channels at boot.
