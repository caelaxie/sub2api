# System update, rollback, and restart

Admins can check for release updates, apply a binary update, roll back to the
previous binary, and restart the service.

```text
admin system action
  |
  +-- idempotency key creates operation id
  +-- acquire system operation lock
  +-- run update, rollback, or restart
  +-- release lock
```

## Update flow

1. The admin checks the current version and latest GitHub release.
2. Results are cached unless the request forces a fresh check.
3. Update selects the release archive matching the current OS and architecture.
4. Download URLs are restricted to trusted GitHub asset hosts.
5. Optional checksums are verified.
6. The new binary is extracted beside the current executable.
7. The current binary is renamed to a backup path.
8. The new binary is atomically renamed into place.
9. The response tells the admin a restart is needed.

## Rollback and restart

1. Rollback restores the backup binary and also requires a restart.
2. Restart schedules a systemd restart after the response is sent so the client
   receives confirmation first.

## Important rules

- Update, rollback, and restart use idempotency plus a system-operation lock so
  duplicate clicks or concurrent admins do not run conflicting operations.
- Source builds may warn differently from release builds.
- Download size and host are validated.
- Restart assumes the process is managed by the expected service mechanism.

## Source map

- `backend/internal/server/routes/admin.go`
- `backend/internal/handler/admin/system_handler.go`
- `backend/internal/service/update_service.go`
- `backend/internal/service/system_operation_lock_service.go`
- `backend/internal/pkg/sysutil/restart.go`
- `frontend/src/components/common/VersionBadge.vue`

