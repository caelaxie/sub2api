# Anthropic-compatible gateway requests

This workflow handles Claude/Anthropic-compatible client calls, mainly
`POST /v1/messages` and token counting. It may forward to Anthropic-compatible
accounts, Gemini compatibility code, or Antigravity depending on account and
group configuration.

```text
/v1/messages
  |
  +-- parse body and model
  +-- detect Claude Code and request features
  +-- acquire user slot
  +-- billing and quota re-check
  +-- choose account
  +-- acquire account slot
  +-- forward upstream
  +-- record usage
```

## Main flow

1. The handler reads the full request body and parses the model, stream flag,
   thinking state, and other request shape.
2. It resolves channel model mapping and restrictions.
3. It detects Claude Code traffic and may enforce minimum supported client
   versions.
4. It checks the user's wait queue and acquires a user concurrency slot.
5. It re-checks subscription, quota, and billing state after waiting.
6. It generates a session hash for sticky routing.
7. It asks the scheduler for an account that can serve the request.
8. It acquires the selected account's concurrency slot.
9. It may intercept known warmup or suggestion-probe requests.
10. It forwards the request to the upstream service.
11. It records usage through the usage worker pool.

## Failover

The handler can switch accounts when upstream errors are retryable and no bytes
have been streamed to the client yet. Once streaming has started, the handler
cannot safely replace the response with another account's response.

## Important rules

- Empty bodies and missing models return bad-request errors.
- User wait queue overflow returns a rate-limit error.
- No schedulable account returns a service-unavailable style error.
- Beta policy, Claude Code restrictions, channel restrictions, and prompt
  length fallback can change the route before forwarding.
- Token counting is not supported for OpenAI groups on `/v1/messages/count_tokens`.

## Source map

- `backend/internal/handler/gateway_handler.go`
- `backend/internal/handler/gateway_handler_responses.go`
- `backend/internal/handler/gateway_handler_chat_completions.go`
- `backend/internal/handler/gateway_helper.go`
- `backend/internal/service/gateway_service.go`
- `backend/internal/service/gemini_messages_compat_service.go`
- `backend/internal/service/antigravity_gateway_service.go`

