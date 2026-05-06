# Admin usage search and cleanup

Admins use usage search to investigate traffic and billing. Cleanup tasks delete
old or unwanted usage logs in controlled batches and then trigger dashboard
aggregation recomputation.

```text
admin usage page
  |
  +-- filter records
  |
  +-- create cleanup task from filters
        |
        +-- worker claims pending task
        +-- delete in batches
        +-- update progress
        +-- recompute dashboard range
```

## Search flow

1. Admins filter usage by date range, user, API key, account, group, model,
   request type, stream flag, and billing type.
2. The admin API can search users and API keys for filter controls.
3. Stats endpoints summarize the current filtered usage.

## Cleanup flow

1. Admin creates a cleanup task from a validated filter set.
2. The task is stored as pending.
3. A timing-wheel worker claims one pending task at a time.
4. The service deletes matching usage logs in batches.
5. Progress is written after each batch.
6. A task can be canceled while pending or running.
7. When the task succeeds, dashboard aggregation is recomputed for the affected
   time range.

## Important rules

- Cleanup requires an authenticated admin subject.
- Start and end date are required.
- The cleanup feature can be disabled by config.
- Invalid date ranges, IDs, request types, and over-large ranges are rejected.
- Cancellation conflicts once a task is already terminal.

## Source map

- `backend/internal/server/routes/admin.go`
- `backend/internal/handler/admin/usage_handler.go`
- `backend/internal/service/usage_cleanup_service.go`
- `backend/internal/repository/usage_cleanup_repo.go`
- `backend/ent/schema/usage_cleanup_task.go`
- `frontend/src/views/admin/UsageView.vue`
- `frontend/src/components/admin/usage/UsageCleanupDialog.vue`
- `frontend/src/components/admin/usage/UsageTable.vue`
