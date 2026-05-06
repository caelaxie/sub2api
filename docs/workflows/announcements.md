# Announcements

Announcements let admins send visible messages to users. They can be popup or
silent, scheduled, targeted, and tracked for read status.

```text
admin creates announcement
  |
  +-- target all users or filtered users
  |
  +-- users fetch unread announcements
  |
  +-- user marks read
```

## Admin flow

1. Admin lists and filters announcements.
2. Admin creates or updates title, content, status, notification mode, and
   schedule.
3. Admin chooses all users or custom targeting rules.
4. Admin can inspect read status for an announcement.

## User flow

1. The frontend fetches announcements on a timed cache.
2. Header and popup components show unread announcements.
3. Popup announcements are queued once per session.
4. The user dismisses or marks an announcement read.

## Important rules

- Title and content are required.
- Schedule start must be before schedule end.
- Targeting can include subscription and balance conditions.
- Invalid announcement IDs are rejected.
- Read failures are logged by the frontend but do not block the rest of the UI.

## Source map

- `backend/internal/server/routes/admin.go`
- `backend/internal/server/routes/user.go`
- `backend/internal/handler/admin/announcement_handler.go`
- `backend/internal/handler/announcement_handler.go`
- `backend/internal/service/announcement.go`
- `frontend/src/views/admin/AnnouncementsView.vue`
- `frontend/src/components/admin/announcements/AnnouncementTargetingEditor.vue`
- `frontend/src/stores/announcements.ts`
- `frontend/src/components/common/AnnouncementBell.vue`
- `frontend/src/components/common/AnnouncementPopup.vue`

