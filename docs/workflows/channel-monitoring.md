# Channel monitoring

Channel monitoring checks configured provider endpoints and models on a schedule.
Admins manage monitors and templates. Users can see status for available
monitors.

```text
monitor config
  |
  +-- runner schedules one ticker per enabled monitor
        |
        +-- worker pool runs HTTP checks
        +-- history and rollups are recorded
        +-- admin/user status views read results
```

## Admin flow

1. Admin creates a monitor with provider, endpoint, API key, models, group,
   interval, and optional template.
2. API keys are encrypted and masked when returned.
3. Admin can run a monitor immediately.
4. Admin can view status, history, and result details.
5. Admin can create templates and apply them to monitors.

## Runner flow

1. Startup loads enabled monitors.
2. Each monitor gets its own ticker.
3. New or updated monitors are scheduled immediately through a scheduler
   callback.
4. The first run happens immediately, then repeats by interval.
5. A worker pool runs actual checks so bursts do not overwhelm upstreams.
6. Duplicate in-flight runs for the same monitor are skipped.

## User flow

1. Users can list channel monitors available to them.
2. Users can inspect status without seeing admin-only secrets.

## Important rules

- Interval must be within the configured range.
- Endpoints must be HTTPS origins and must not target localhost, private
  networks, or metadata services.
- Decrypt failures are surfaced instead of silently using broken credentials.
- If the worker pool is full, that run is skipped.

## Source map

- `backend/internal/server/routes/admin.go`
- `backend/internal/server/routes/user.go`
- `backend/internal/handler/admin/channel_monitor_handler.go`
- `backend/internal/handler/admin/channel_monitor_template_handler.go`
- `backend/internal/handler/channel_monitor_user_handler.go`
- `backend/internal/service/channel_monitor_service.go`
- `backend/internal/service/channel_monitor_runner.go`
- `backend/internal/service/channel_monitor_validate.go`
- `frontend/src/views/admin/ChannelMonitorView.vue`
- `frontend/src/views/user/ChannelStatusView.vue`

