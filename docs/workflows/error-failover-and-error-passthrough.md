# Error failover and error passthrough

Gateway errors can trigger account failover, operator-visible ops records, or
custom passthrough responses. The exact behavior depends on whether response
streaming has already started and whether an admin configured passthrough rules.

```text
upstream error
  |
  +-- retryable and no stream bytes sent?
  |      |
  |      +-- try next account
  |
  +-- failover exhausted
         |
         +-- match passthrough rule?
         |      +-- write configured status/body
         |
         +-- otherwise map to protocol error
```

## Main flow

1. The upstream forwarding service returns structured errors when a request can
   be retried on another account.
2. The handler decides whether it is still safe to fail over. Failover is unsafe
   after bytes have been written to a streaming response.
3. If failover is safe, the handler excludes the failed account and asks the
   scheduler for another account.
4. After failover is exhausted, error passthrough rules may match platform,
   status code, headers, and body keywords.
5. A matching rule can override status/body and can decide whether monitoring
   should record the error.
6. If no rule matches, the handler maps the upstream error into Anthropic,
   OpenAI, or Google error format.

## Important rules

- 401 and 403 upstream responses usually become gateway-side 502 behavior.
- 429 usually remains rate-limit behavior.
- 529 maps to service-unavailable behavior.
- Passthrough rules are admin-managed and cached.
- Ops middleware and handlers attach request/error context for later triage.

## Source map

- `backend/internal/service/error_passthrough_service.go`
- `backend/internal/service/error_passthrough_runtime.go`
- `backend/internal/handler/openai_gateway_handler.go`
- `backend/internal/handler/gateway_handler.go`
- `backend/internal/handler/gemini_v1beta_handler.go`
- `backend/internal/handler/ops_error_logger.go`
- `backend/internal/handler/admin/error_passthrough_handler.go`
- `frontend/src/components/admin/ErrorPassthroughRulesModal.vue`

