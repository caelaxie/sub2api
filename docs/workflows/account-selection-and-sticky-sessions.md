# Account selection and sticky sessions

Account selection decides which upstream account will serve a gateway request.
It combines group policy, platform, model support, sticky sessions, concurrency,
quota, RPM, temporary unschedulable state, and load awareness.

```text
request wants model X
  |
  +-- resolve group and platform
  +-- build candidate accounts
  +-- try sticky account when valid
  +-- filter by model, quota, RPM, session, state
  +-- acquire account slot or wait plan
  +-- return selected account
```

## Main flow

1. Resolve the API key's group and platform.
2. Check group restrictions such as Claude Code restrictions, OAuth-only
   requirements, and channel restrictions.
3. Read channel model routing to know whether the requested model maps to a
   specific upstream model or account set.
4. Build the candidate account list from group bindings and platform rules.
5. Prefetch window-cost and RPM state so filtering does not issue one query per
   account.
6. Try the sticky account first if a session hash exists.
7. Skip accounts that are disabled, temporarily unschedulable, model-restricted,
   over quota, over RPM, over session limit, or unable to acquire a concurrency
   slot.
8. Acquire an account slot or return a wait plan.
9. Hydrate credentials before forwarding.

## Sticky sessions

Sticky sessions keep related client turns on the same upstream account when the
account is still valid. Sub2API uses request-derived session hashes and Redis
caches to bind sessions to account IDs.

Sticky routing is helpful for clients that rely on upstream conversation state,
but it is not absolute. The scheduler can skip a sticky account if it is no
longer safe or allowed.

## Important rules

- No valid candidates returns a no-available-account error.
- Redis wait-count errors generally fail open so requests are not blocked by a
  transient cache issue.
- Sticky accounts are skipped when they violate current policy or runtime state.
- Mixed scheduling can allow multiple platforms only when group/channel policy
  permits it.

## Source map

- `backend/internal/service/gateway_service.go`
- `backend/internal/service/openai_gateway_service.go`
- `backend/internal/service/gemini_messages_compat_service.go`
- `backend/internal/service/concurrency_service.go`
- `backend/internal/service/sticky_session_test.go`
- `backend/internal/service/scheduler_snapshot_service.go`
- `backend/internal/service/scheduler_events.go`

