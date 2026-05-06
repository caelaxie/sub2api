# OpenAI gateway requests

OpenAI gateway paths handle OpenAI-native Responses, Chat Completions, image
generation/editing, OpenAI-compatible Anthropic Messages, and Responses
WebSocket traffic.

```text
OpenAI group request
  |
  +-- route selects OpenAI handler
  |
  +-- validate request shape
  |
  +-- acquire slots and billing permission
  |
  +-- select OpenAI account
  |
  +-- forward, retry, record usage
```

## HTTP Responses, Chat Completions, and Images

1. The handler validates JSON, model, stream flag, and endpoint-specific fields.
2. HTTP Responses rejects unsupported continuation cases such as
   `previous_response_id` when WebSocket v2 is required.
3. Image generation and edit requests apply an image concurrency limit.
4. The handler acquires a user slot and re-checks billing/quota.
5. It computes or reads session information.
6. The scheduler selects an OpenAI account.
7. The service forwards the request and may retry within the selected pool or
   switch accounts when safe.
8. Usage is submitted to the usage worker pool.

## Anthropic Messages on OpenAI accounts

When an OpenAI group receives `/v1/messages`, Sub2API can convert the
Anthropic-style request into OpenAI/Codex-compatible upstream calls, then convert
the response back into Anthropic shape.

This path is controlled by group policy. If messages dispatch is disabled for
the group, the request is rejected.

## Responses WebSocket

1. The client must send a WebSocket upgrade.
2. The first message must be a valid `response.create`.
3. Initial user/account slots and billing checks run before proxying.
4. The relay keeps the socket open but holds concurrency only during active
   turns.
5. Usage is recorded per completed turn.

## Important rules

- Missing WebSocket upgrade returns upgrade-required behavior.
- Invalid first WebSocket messages close the socket.
- Message IDs passed as response IDs are rejected.
- Function-call output requests need call context.
- Compact requests need compact-capable accounts.

## Source map

- `backend/internal/handler/openai_gateway_handler.go`
- `backend/internal/handler/openai_chat_completions.go`
- `backend/internal/handler/openai_images.go`
- `backend/internal/service/openai_gateway_service.go`
- `backend/internal/service/openai_ws_v2/entry.go`
- `backend/internal/service/openai_ws_v2/passthrough_relay.go`
- `backend/internal/pkg/apicompat/*`

