# Admin groups, channels, and policy

Groups and channels define how users and API keys reach upstream accounts.
Groups hold policy. Channels attach routing, model mapping, platform behavior,
and pricing to groups.

```text
admin policy setup
  |
  +-- create group
  |      defines platform, billing, subscription, limits
  |
  +-- create channel
         assigns group IDs and model/pricing behavior
```

## Group flow

1. Admins create or update a group.
2. Group fields define platform, billing mode, subscription policy, fallback,
   model routing, OAuth-only requirements, privacy requirements, rate
   multipliers, and RPM overrides.
3. Groups can be attached to users, API keys, accounts, and subscription plans.
4. Admins inspect group usage and capacity.

## Channel flow

1. Admins create a channel and assign group IDs.
2. The channel enables one or more platform configs.
3. Model mapping can rewrite a requested model to an upstream model.
4. Restricted models can block unsupported requests.
5. Pricing can be configured for tokens, requests, images, and account-stat
   charging rules.

## Important rules

- A group can belong to only one channel.
- Subscription assignment requires a subscription-type group.
- OAuth-only groups reject API-key accounts.
- Pricing intervals cannot overlap.
- Channel platform matching is strict and affects gateway routing.

## Source map

- `backend/internal/server/routes/admin.go`
- `backend/internal/handler/admin/group_handler.go`
- `backend/internal/handler/admin/channel_handler.go`
- `backend/internal/service/channel_service.go`
- `backend/internal/service/group_capacity_service.go`
- `frontend/src/views/admin/GroupsView.vue`
- `frontend/src/views/admin/ChannelsView.vue`
- `frontend/src/components/admin/group/*`
- `frontend/src/components/admin/channel/*`

