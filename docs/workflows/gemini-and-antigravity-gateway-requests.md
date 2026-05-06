# Gemini and Antigravity gateway requests

Gemini-native routes support Gemini SDK/CLI traffic on `/v1beta`. Antigravity
routes force Antigravity account selection while reusing much of the gateway
and compatibility machinery.

```text
/v1beta/models/{model}:{action}
  |
  +-- Google-style API key auth
  +-- parse model and action
  +-- map/restrict model by channel
  +-- acquire user slot
  +-- select Gemini or Antigravity account
  +-- forward native request
  +-- record usage
```

## Gemini-native flow

1. The `/v1beta` route uses Google-style auth and error writing.
2. The handler parses the model/action segment. Examples include generate,
   stream generate, and token counting actions.
3. Channel mapping can rewrite the requested model to an upstream model.
4. Billing and user concurrency checks run before account selection.
5. The handler derives Gemini CLI session information or falls back to request
   digest session logic.
6. The Gemini compatibility service selects an account and forwards the native
   request.
7. Streaming and non-streaming responses are translated as needed.
8. Long-context usage can affect billing rules.

## Antigravity forced routes

1. `/antigravity/v1` and `/antigravity/v1beta` apply a force-platform
   middleware.
2. The forced platform prevents normal mixed scheduling from choosing other
   account types.
3. Antigravity-specific token provider and quota paths are used.

## Important rules

- A bad `{model}:{action}` path returns not-found behavior.
- Unsupported actions return not-found behavior.
- Wrong platform or missing accounts returns protocol-specific errors.
- When a request moves to a different Gemini account, stored thought signatures
  may be cleaned to avoid upstream errors.
- Gemini long-context billing has special threshold and multiplier behavior.

## Source map

- `backend/internal/server/routes/gateway.go`
- `backend/internal/handler/gemini_v1beta_handler.go`
- `backend/internal/service/gemini_messages_compat_service.go`
- `backend/internal/service/antigravity_gateway_service.go`
- `backend/internal/service/gemini_token_provider.go`
- `backend/internal/service/antigravity_token_provider.go`
- `backend/internal/pkg/geminicli/*`
- `backend/internal/pkg/antigravity/*`

