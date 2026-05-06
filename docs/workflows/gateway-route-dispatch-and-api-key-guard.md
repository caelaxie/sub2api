# Gateway route dispatch and API key guard

Gateway routes accept client traffic that looks like Anthropic, OpenAI, Gemini,
or Antigravity APIs. Route dispatch decides which handler gets the request, and
API key middleware decides whether the request is allowed to reach a handler.

```text
client request
  |
  +-- route group applies common middleware
  |
  +-- API key auth loads key/user/group/subscription
  |
  +-- route dispatch picks protocol handler
```

## Route dispatch

1. `/v1` handles Claude/Anthropic-style routes and OpenAI-compatible aliases.
2. `/v1/messages`, `/v1/responses`, and `/v1/chat/completions` can branch to
   OpenAI handlers when the API key group is an OpenAI group.
3. `/v1beta` handles Gemini-native SDK/CLI endpoints with Google-style errors.
4. `/antigravity/v1` and `/antigravity/v1beta` force Antigravity platform
   routing instead of mixing with normal group platform selection.
5. Non-OpenAI groups get 404 responses for OpenAI image endpoints.

## API key guard

1. Middleware extracts the API key from headers.
2. Query-string API keys are rejected.
3. The API key is loaded and validated.
4. The user, role, group, and subscription are attached to request context.
5. IP allow/deny, key status, expiration, balance, quota, and rate limits are
   checked before the handler runs.

## Important rules

- `/v1/usage` still authenticates but skips normal billing enforcement.
- Missing or invalid keys return authentication errors.
- Disabled users, expired keys, denied IPs, exhausted quota, or insufficient
  balance are rejected before upstream work starts.
- Group assignment may be required. If a key has no allowed group, the error is
  written in the protocol style of that route.

## Source map

- `backend/internal/server/routes/gateway.go`
- `backend/internal/server/middleware/api_key_auth.go`
- `backend/internal/server/middleware/api_key_auth_google.go`
- `backend/internal/server/middleware/middleware.go`
- `backend/internal/handler/endpoint.go`

