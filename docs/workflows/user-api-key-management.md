# User API key management

Users create and manage API keys that allow clients to call gateway endpoints.
Each key belongs to the user and may be tied to a group, quota, rate limits,
expiration, IP rules, and status.

```text
user dashboard
  |
  +-- load keys, groups, rates, usage stats
  |
  +-- create/update/delete key
        |
        +-- backend checks ownership
        +-- cache and rate-limit data are invalidated as needed
```

## Main flow

1. The keys page loads the user's keys.
2. It also loads available groups, group rates, public settings, and usage
   stats so the form can show the correct choices.
3. Create sends name, group, optional custom key, IP allow/deny lists, quota,
   expiration, and rate limits.
4. Update can change the key metadata, status, group, quota, expiration, or rate
   limit settings.
5. Delete removes the key.
6. The backend enforces that the authenticated user owns the key for get, update,
   and delete.

## Important rules

- Invalid key IDs are rejected.
- Accessing another user's key returns an authorization error.
- Invalid `expires_at` values are rejected.
- Create is coordinated through idempotency so duplicate submissions do not
  create duplicate keys.
- Group selection matters because gateway routing and billing later read the
  key's group.

## Source map

- `backend/internal/server/routes/user.go`
- `backend/internal/handler/api_key_handler.go`
- `backend/internal/service/api_key.go`
- `frontend/src/views/user/KeysView.vue`
- `frontend/src/api/keys.ts`
- `frontend/src/components/keys/UseKeyModal.vue`
- `frontend/src/components/keys/EndpointPopover.vue`

