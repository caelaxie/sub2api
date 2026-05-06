# Ops dashboard, errors, and runtime logging

Ops workflows help admins understand live traffic, diagnose failures, retry
stored requests, tune runtime logging, and clean indexed logs.

```text
gateway request
  |
  +-- ops middleware records context on errors
  |
  +-- ops services aggregate metrics
  |
  +-- admin ops UI reads dashboards and drilldowns
```

## Dashboard and realtime flow

1. Metrics collectors and aggregators record throughput, latency, concurrency,
   account availability, token stats, and error trends.
2. Admin dashboard endpoints return overview charts, histograms, distributions,
   and realtime traffic.
3. WebSocket endpoints stream selected realtime signals.

## Error triage flow

1. Gateway and ops middleware attach request and upstream error context.
2. Admin lists request errors, upstream errors, and stored request details.
3. Admin can drill into a request or upstream error.
4. Admin can retry a stored client request or an upstream attempt when the stored
   body is available.
5. Admin can mark errors resolved.

## Runtime logging and cleanup

1. Runtime alert and logging settings are stored in settings.
2. Runtime log config can be reset to defaults.
3. Indexed system logs can be searched and cleaned by filter.
4. Cleanup is irreversible and should be run with narrow filters.

## Important rules

- Ops can be disabled by config or settings.
- Time windows and page sizes are capped.
- Retry requires an admin subject and stored request/upstream body.
- Some scheduled ops work uses Redis leader locks so only one instance runs it.

## Source map

- `backend/internal/server/routes/admin.go`
- `backend/internal/handler/admin/ops_handler.go`
- `backend/internal/handler/admin/ops_dashboard_handler.go`
- `backend/internal/handler/admin/ops_system_log_handler.go`
- `backend/internal/handler/admin/ops_settings_handler.go`
- `backend/internal/handler/admin/ops_ws_handler.go`
- `backend/internal/service/ops_service.go`
- `backend/internal/service/ops_aggregation_service.go`
- `backend/internal/service/ops_alerts.go`
- `backend/internal/service/ops_system_log_sink.go`
- `frontend/src/views/admin/ops/OpsDashboard.vue`

