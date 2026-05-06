# Backups and deprecated data management

The current backup workflow is an in-process admin backup service. Older
data-management agent routes still exist, but the service now reports that path
as deprecated.

```text
admin backup page
  |
  +-- configure S3
  +-- optional cron schedule
  +-- create backup
        |
        +-- pg_dump
        +-- gzip
        +-- upload to S3
        +-- record status in settings
```

## Backup flow

1. Admin configures S3-compatible storage.
2. Admin can test the S3 connection.
3. Admin can configure a cron schedule and retention policy.
4. Manual or scheduled backup starts a background operation.
5. The service runs the database dumper, compresses output with gzip, and uploads
   it to object storage.
6. Records are stored in settings.
7. Startup marks stale running backup or restore records as failed.

## Restore flow

1. Admin selects a completed backup.
2. Admin re-enters password before restore.
3. The service downloads the S3 object, decompresses it, and restores through the
   database dumper.
4. Restore status is written back to the backup record.

## Deprecated data-management agent

The `/admin/data-management` routes still exist for compatibility, but the
current service returns `DATA_MANAGEMENT_DEPRECATED` for non-health operations.
New code should use the in-process backup service instead of the old
`datamanagementd` agent path.

## Important rules

- Only one backup or restore can run at a time.
- S3 config is required for backup storage.
- Only completed backups can be downloaded or restored.
- The deprecated data-management health endpoint reports disabled/deprecated
  state.

## Source map

- `backend/internal/server/routes/admin.go`
- `backend/internal/handler/admin/backup_handler.go`
- `backend/internal/service/backup_service.go`
- `backend/internal/repository/backup_pg_dumper.go`
- `backend/internal/service/data_management_service.go`
- `backend/internal/handler/admin/data_management_handler.go`
- `frontend/src/views/admin/BackupView.vue`
- `deploy/DATAMANAGEMENTD_CN.md`

