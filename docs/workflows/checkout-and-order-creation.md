# Checkout and order creation

Authenticated users create balance or subscription orders. The backend validates
the order, creates a pending order, calls the selected provider, and returns
provider-specific checkout data.

```text
user checkout
  |
  +-- load config, plans, channels, limits
  |
  +-- POST /payment/orders
        |
        +-- validate amount/plan/user
        +-- select provider
        +-- create PENDING order
        +-- call provider
        +-- return QR, URL, or Stripe session data
```

## Main flow

1. The payment page loads payment config, checkout info, plans, channels, and
   limits.
2. The user chooses balance recharge or subscription purchase.
3. The backend validates the user is active and payment is enabled.
4. It checks pending-order and daily amount limits.
5. It calculates order amount, fee, payable amount, and provider selection.
6. It creates a `PENDING` payment order in a transaction.
7. It invokes the provider to create the upstream payment.
8. It returns provider-specific data such as QR code URL, redirect URL, or Stripe
   checkout session information.

## WeChat payment OAuth note

Official WeChat Pay may require a payment OAuth flow before order creation can
finish. In that case, the backend returns a response that tells the frontend to
start WeChat payment OAuth and resume checkout afterward.

## Important rules

- `PAYMENT_DISABLED`, `TOO_MANY_PENDING`, inactive user, daily-limit exceeded,
  unsupported method, and `NO_AVAILABLE_INSTANCE` stop order creation.
- Provider create failure marks the order failed.
- Public resume/verify endpoints exist so a payment result page can recover
  order state after redirect flows.

## Source map

- `backend/internal/server/routes/payment.go`
- `backend/internal/handler/payment_handler.go`
- `backend/internal/service/payment_order.go`
- `backend/internal/service/payment_resume_service.go`
- `frontend/src/views/user/PaymentView.vue`
- `frontend/src/views/user/PaymentQRCodeView.vue`
- `frontend/src/views/user/StripePaymentView.vue`
- `frontend/src/views/user/PaymentResultView.vue`
- `frontend/src/components/payment/*`

