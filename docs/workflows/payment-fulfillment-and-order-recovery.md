# Payment fulfillment and order recovery

Payment fulfillment turns a provider payment into Sub2API balance or
subscription entitlement. The same service also protects against late payments,
cancel races, expiry races, and failed fulfillment retries.

```text
provider callback or user verify
  |
  +-- verify provider signature/status
  +-- match payment order
  +-- mark order paid
  +-- fulfill balance or subscription
  +-- write audit log
```

## Webhook fulfillment

1. Provider webhook routes receive public callbacks.
2. The webhook handler verifies provider signatures or provider-specific
   callback format.
3. The service matches the notification to an order.
4. Unknown successful notifications are acknowledged so providers do not retry
   forever.
5. Amount, provider, and metadata are validated.
6. The order moves to paid status.
7. Fulfillment applies balance recharge or subscription assignment.
8. Audit logs record the order status and fulfillment actions.

## Expiry, cancel, verify, and retry

1. A background worker expires timed-out pending orders.
2. Before expiring or cancelling, the service can query the provider to avoid
   losing a payment that succeeded upstream.
3. Recent expired orders have a short recovery window.
4. Users can verify orders after checkout.
5. Admins can retry failed fulfillment for eligible paid orders.
6. User and admin cancel paths only work on pending orders.

## Important rules

- Signature failures return bad request.
- Amount mismatch above the tolerance is rejected.
- Cancel can be rate limited.
- Retry does not apply to completed, recharging, refunded, or invalid states.
- Fulfillment is designed to be idempotent so callbacks and manual retries do
  not double-credit the user.

## Source map

- `backend/internal/server/routes/payment.go`
- `backend/internal/handler/payment_webhook_handler.go`
- `backend/internal/service/payment_fulfillment.go`
- `backend/internal/service/payment_order_lifecycle.go`
- `backend/internal/service/payment_order_expiry_service.go`
- `backend/internal/service/payment_order.go`
- `backend/internal/payment/provider/*`
- `frontend/src/views/user/PaymentResultView.vue`
- `frontend/src/views/admin/orders/AdminOrdersView.vue`
