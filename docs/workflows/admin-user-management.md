# Admin user management

Admins manage users, inspect their usage, bind identities, adjust balances, and
edit per-user attributes.

```text
admin user table
  |
  +-- filter/search users
  +-- create or update user
  +-- inspect keys, usage, balance, RPM
  +-- adjust balance or group
  +-- edit attributes or identities
```

## Main flow

1. The admin user page lists users with filters for status, role, search terms,
   groups, and attributes.
2. Admins can create users with email, password, role, status, balance, and
   default limits.
3. Admins can update role, status, limits, group access, and profile details.
4. Balance updates use explicit operations such as set, add, or subtract.
5. Admins can inspect a user's API keys, usage, balance history, RPM status, and
   custom attributes.
6. Admins can bind auth identities to a user.

## Important rules

- Invalid user IDs and invalid filters are rejected.
- Passwords must pass backend validation.
- Balance updates must use a supported operation.
- Identity binding is explicit about provider and channel.
- User attributes are defined separately and then assigned per user.

## Source map

- `backend/internal/server/routes/admin.go`
- `backend/internal/handler/admin/user_handler.go`
- `backend/internal/handler/admin/user_attribute_handler.go`
- `backend/internal/service/admin_service.go`
- `backend/internal/service/user_attribute_service.go`
- `frontend/src/views/admin/UsersView.vue`
- `frontend/src/components/admin/user/*`

