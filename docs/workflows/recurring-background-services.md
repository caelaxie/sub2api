# Recurring background services

Several services run in the background after normal application startup. They
keep tokens fresh, expire stale records, aggregate metrics, run scheduled tests,
send queued email, clean old data, and stop cleanly during shutdown.

```text
normal startup
  |
  +-- Wire builds services
  +-- provider functions call Start()
  +-- HTTP server runs
  |
  +-- shutdown signal
        |
        +-- HTTP graceful shutdown
        +-- cleanup function stops workers and closes infra
```

## Background services

- Token refresh checks active OAuth accounts and refreshes near-expiry tokens.
- Account expiry and subscription expiry run every minute.
- Payment order expiry marks timed-out pending orders after checking provider
  state where needed.
- Scheduled account tests run due plans every minute.
- Channel monitor runs one scheduled task per enabled monitor.
- Usage cleanup deletes old usage logs in batches from queued cleanup tasks.
- Idempotency cleanup removes expired idempotency records.
- Ops metrics, aggregation, alerts, cleanup, reports, and log sink support the
  ops dashboard.
- Dashboard aggregation recomputes dashboard summary data.
- Email queue sends verification and password-reset emails with worker
  concurrency.
- Deferred account last-used updates batch frequent writes.
- Scheduler snapshot service keeps account/group scheduling caches fresh.

## Shutdown flow

1. The main server receives SIGINT or SIGTERM.
2. The HTTP server gets a graceful shutdown timeout.
3. The application cleanup function stops background services.
4. Several independent services stop in parallel.
5. Infrastructure clients such as Redis and Ent are closed.
6. Cleanup logs failures but continues through the remaining steps.

## Important rules

- Some services are disabled by config or missing dependencies.
- Ops cleanup and reports use Redis leader locks in multi-instance deployment.
- Channel monitor skips duplicate in-flight runs and full worker pools.
- Email queue can be full.
- Cleanup has timeout handling so shutdown does not wait forever.

## Source map

- `backend/cmd/server/main.go`
- `backend/cmd/server/wire.go`
- `backend/internal/service/wire.go`
- `backend/internal/service/token_refresh_service.go`
- `backend/internal/service/account_expiry_service.go`
- `backend/internal/service/subscription_expiry_service.go`
- `backend/internal/service/payment_order_expiry_service.go`
- `backend/internal/service/scheduled_test_runner_service.go`
- `backend/internal/service/channel_monitor_runner.go`
- `backend/internal/service/usage_cleanup_service.go`
- `backend/internal/service/idempotency_cleanup_service.go`
- `backend/internal/service/email_queue_service.go`
- `backend/internal/service/deferred_service.go`
- `backend/internal/service/scheduler_snapshot_service.go`

