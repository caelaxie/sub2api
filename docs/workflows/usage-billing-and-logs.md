# Usage billing and logs

Sub2API records gateway usage and charges the correct quota or balance after a
request completes. The handler captures metadata, submits a task, and the
service writes usage and billing data.

```text
gateway response completed
  |
  +-- usage task submitted
  |
  +-- calculate cost
  |
  +-- charge subscription or balance
  |
  +-- write usage log
  |
  +-- update caches and stats
```

## Main flow

1. Gateway handlers capture user, API key, account, group, endpoint, request
   type, model, token usage, image usage, and timing data.
2. Usage tasks are submitted to a bounded worker pool.
3. Pricing resolves model and channel-specific rates.
4. User/group multipliers are applied.
5. The billing path decides whether to charge subscription quota, account quota,
   user balance, or simple-mode record-only behavior.
6. Idempotent billing commands prevent double-charging the same request.
7. Usage logs are written for dashboards, user history, admin search, and
   cleanup tasks.
8. Billing and API key caches are updated or invalidated.

## Important rules

- Simple mode records usage without normal billing checks.
- Billing repository conflicts protect against duplicate charges.
- Billing cache failures can trip service-unavailable behavior.
- RPM checks can return rate-limit responses with retry timing.
- Usage logs are later used by dashboards, admin cleanup, account stats, and
  quota windows.

## Source map

- `backend/internal/service/usage_billing.go`
- `backend/internal/service/usage_service.go`
- `backend/internal/service/usage_record_worker_pool.go`
- `backend/internal/service/billing_service.go`
- `backend/internal/service/billing_cache_service.go`
- `backend/internal/repository/usage_log_repo.go`
- `backend/ent/schema/usage_log.go`
- `backend/internal/handler/usage_handler.go`
- `backend/internal/handler/admin/usage_handler.go`
