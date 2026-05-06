# Subscriptions

Subscriptions connect users to subscription groups and quota windows. They can
be granted by admin action, payment fulfillment, or redeem codes.

```text
grant source
  |
  +-- admin assign / payment / redeem
        |
        +-- user_subscription row
        +-- billing cache invalidation
        +-- gateway quota checks use active subscription
```

## User flow

1. `/subscriptions` lists all subscriptions for the current user.
2. `/subscriptions/active` lists active subscriptions.
3. `/subscriptions/progress` calculates current usage against active quota.
4. `/subscriptions/summary` returns a compact dashboard summary.
5. The frontend store caches active subscriptions and deduplicates concurrent
   requests.

## Admin flow

1. Admins assign a subscription group to a user.
2. Admins can bulk assign users.
3. Admins can extend or shorten validity.
4. Admins can reset one or more quota windows.
5. Admins can revoke a subscription.

## Important rules

- Admin assignment requires a subscription-type group.
- Validity has a maximum day limit.
- Reset quota requires at least one quota window.
- The progress endpoint skips a subscription if its progress lookup fails.
- Active subscription state is refreshed after payment or redeem actions.

## Source map

- `backend/internal/server/routes/user.go`
- `backend/internal/server/routes/admin.go`
- `backend/internal/handler/subscription_handler.go`
- `backend/internal/handler/admin/subscription_handler.go`
- `backend/internal/service/subscription_service.go`
- `frontend/src/views/user/SubscriptionsView.vue`
- `frontend/src/views/admin/SubscriptionsView.vue`
- `frontend/src/stores/subscriptions.ts`

