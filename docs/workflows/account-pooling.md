# Account pooling

Account pooling is the way Sub2API chooses one upstream account from the set of
accounts available to a group. It is separate from the per-account pool mode
flag, which is used when one configured upstream credential already represents
an external account pool.

```text
gateway request
  |
  +-- resolve group, platform, model, and session
  +-- collect accounts bound to that group and platform
  +-- remove accounts that cannot safely serve the request
  +-- prefer model-routed accounts when configured
  +-- reuse a sticky account when it is still valid
  +-- choose the least-loaded eligible account
  +-- acquire a slot, wait, or fail with no available accounts
```

## Normal account pool

The normal pool is made from local account records. Accounts carry scheduling
fields such as platform, type, concurrency, priority, load factor, status,
schedulable state, quota state, rate-limit state, and group bindings.

When a gateway request arrives, the scheduler resolves the request group and
platform, then lists accounts that are schedulable for that group. From there it
filters by current policy and runtime state:

1. The account must be active and schedulable.
2. The account must match the requested platform, unless mixed scheduling is
   allowed.
3. The account must support the requested model or model mapping.
4. The account must pass quota, window-cost, RPM, and model-scope checks.
5. The account must not be temporarily unschedulable, overloaded, expired, or
   excluded by a previous failover attempt.
6. The account must have a free concurrency slot, or the request must be able to
   enter an allowed wait queue.

If no account survives these checks, the request receives a no-available-account
error.

## Model routing

Model routing narrows the pool before general load balancing. When a group maps
a requested model to specific account IDs, those routed accounts are checked
first. The scheduler still applies the same safety checks, so model routing does
not force traffic to an unhealthy, over-quota, unsupported, or full account.

If every routed account is unavailable, the scheduler can fall back to normal
selection.

## Sticky sessions

Sticky sessions bind a request-derived session hash to a selected account. This
keeps related turns on the same upstream account when the account is still safe
to use.

Sticky routing is a preference, not a guarantee. The scheduler can skip or clear
the sticky binding if the account no longer passes platform, model, quota, RPM,
window-cost, session-limit, or runtime-state checks. If the sticky account is
valid but full, the request may wait on that account if the sticky wait queue is
not full.

## Load-aware selection

After filtering, the scheduler asks the concurrency service for account load
state. Eligible accounts are ordered by:

1. Lower priority value.
2. Lower load rate.
3. Older or empty last-used time.

Accounts that tie on those fields are shuffled within the tied group. The
scheduler then tries to acquire a slot in that order. When it acquires a slot,
it registers the session and stores a sticky binding when a session hash exists.

If every eligible account is full, the scheduler returns a wait plan for the
best eligible account instead of picking an unsafe account.

## Pool mode

Pool mode is a different feature. It applies to one local API-key or Bedrock
account whose upstream credential points at an external account pool.

When pool mode is enabled, selected upstream failures do not immediately mark
the local account as bad. Instead, retryable statuses retry on the same local
account first:

```text
selected local account has pool_mode=true
  |
  +-- upstream returns 401, 403, or 429
  +-- retry on the same local account
  +-- stop after the configured retry count
  +-- then continue normal failover if needed
```

The default same-account retry count is 3. A configured value below 0 becomes 0,
and a value above 10 is capped at 10.

OpenAI pool-mode handling also ensures there is a session hash. If the client
did not provide one, Sub2API creates an internal pool-retry session hash so the
same-account retry does not get load-balanced to a different local account.

## Important rules

- Normal account pooling balances across Sub2API account records.
- Pool mode does not expand one local account into many local records.
- Pool mode means the upstream behind that one local account is already a pool.
- Sticky sessions can keep traffic on an account, but only while policy and
  runtime checks still pass.
- Failover excludes failed local accounts during the retry chain. Pool mode
  delays that switch for retryable upstream statuses.

## Source map

- `backend/internal/service/account.go`
- `backend/internal/service/gateway_service.go`
- `backend/internal/service/openai_gateway_service.go`
- `backend/internal/service/concurrency_service.go`
- `backend/internal/handler/openai_gateway_handler.go`
- `backend/internal/handler/openai_chat_completions.go`
- `backend/internal/handler/openai_images.go`
- `frontend/src/components/account/CreateAccountModal.vue`
- `frontend/src/components/account/EditAccountModal.vue`
