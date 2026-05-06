# Scheduled account tests

Scheduled account tests periodically test a specific account/model and keep a
bounded result history. They are used to detect broken upstream accounts and can
optionally recover accounts after a successful test.

```text
admin creates plan
  |
  +-- runner wakes every minute
        |
        +-- find due plans
        +-- run account test
        +-- save bounded result history
        +-- optionally recover account state
```

## Main flow

1. Admin opens an account's scheduled tests panel.
2. Admin creates a plan with model ID, cron expression, enabled flag, max result
   count, and optional auto-recover.
3. The runner wakes every minute and asks for due plans.
4. Each due plan runs an account background test.
5. The result is saved and old results are trimmed to the configured maximum.
6. If the result succeeds and auto-recover is enabled, recoverable error or
   rate-limit state can be cleared.
7. The next run time is computed from the cron expression.

## Important rules

- Cron expressions are five-field expressions.
- Create defaults to enabled and a bounded result count.
- Invalid account IDs and plan IDs are rejected.
- The runner limits concurrent test work.
- Invalid timezone configuration can prevent the runner from using the intended
  location.

## Source map

- `backend/internal/server/routes/admin.go`
- `backend/internal/handler/admin/scheduled_test_handler.go`
- `backend/internal/service/scheduled_test_service.go`
- `backend/internal/service/scheduled_test_runner_service.go`
- `backend/internal/service/account_test_service.go`
- `frontend/src/components/admin/account/ScheduledTestsPanel.vue`

