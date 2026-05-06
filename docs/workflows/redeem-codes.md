# Redeem codes

Redeem codes let an authenticated user apply a prepaid benefit. A code can add
balance, change concurrency, or grant a subscription-like entitlement depending
on how the admin generated it.

```text
user enters code
  |
  +-- POST /api/v1/redeem
        |
        +-- validate code
        +-- apply benefit
        +-- record usage
        +-- refresh user/subscription state in UI
```

## Main flow

1. The redeem page loads recent redeem history.
2. The user submits a code.
3. The backend validates that the code exists and can be used by this user.
4. The service applies the code's benefit.
5. The frontend refreshes the current user after success.
6. If the result is a subscription, the frontend also refreshes active
   subscription state.
7. History is reloaded after redemption.

## Important rules

- Empty codes are blocked before submit.
- The backend requires an authenticated user.
- The user history endpoint returns a limited recent history.
- Subscription-like results affect later gateway quota and billing checks.

## Source map

- `backend/internal/server/routes/user.go`
- `backend/internal/handler/redeem_handler.go`
- `backend/internal/service/redeem_service.go`
- `frontend/src/views/user/RedeemView.vue`
- `frontend/src/api/redeem.ts`

