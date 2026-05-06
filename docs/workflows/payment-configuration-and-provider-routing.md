# Payment configuration and provider routing

Payment configuration is the admin workflow that makes checkout available.
Admins enable payment, configure plans and visible methods, and add provider
instances for EasyPay, Alipay, WeChat Pay, or Stripe.

```text
admin payment setup
  |
  +-- enable payment and limits
  +-- create subscription plans
  +-- create provider instances
  +-- provider router selects instance during checkout
```

## Main flow

1. Admins enable payment and configure global limits, fees, refund behavior,
   visible channels, and checkout options.
2. Admins create subscription plans when subscription purchases are needed.
3. Admins create provider instances with encrypted provider configuration.
4. Provider config is validated for provider-specific required fields.
5. During checkout, the payment load balancer chooses an enabled provider
   instance by the configured strategy, such as round robin or least amount.

## Important rules

- Payment disabled blocks checkout.
- Provider instances with pending orders cannot be safely disabled, deleted, or
  changed in protected ways.
- Missing provider routes produce method/provider availability errors.
- Provider secrets are encrypted and masked when returned to the frontend.

## Source map

- `backend/internal/server/routes/payment.go`
- `backend/internal/handler/admin/payment_handler.go`
- `backend/internal/service/payment_config_service.go`
- `backend/internal/service/payment_config_providers.go`
- `backend/internal/payment/load_balancer.go`
- `backend/internal/payment/provider/*`
- `docs/PAYMENT.md`
- `frontend/src/views/admin/orders/AdminPaymentDashboardView.vue`
- `frontend/src/views/admin/orders/AdminPaymentPlansView.vue`
- `frontend/src/components/admin/payment/*`

