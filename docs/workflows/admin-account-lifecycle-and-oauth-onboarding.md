# Admin account lifecycle and OAuth onboarding

Accounts are upstream credentials. Admins create, edit, test, refresh, recover,
import, export, and bulk update them. OAuth account onboarding is part of this
workflow because OAuth creates or refreshes account credentials.

```text
admin account page
  |
  +-- create API-key or OAuth account
  +-- bind groups and proxy
  +-- test account
  +-- recover or refresh runtime state
  +-- scheduler uses account for gateway traffic
```

## Account lifecycle

1. Admins list accounts with filters for platform, type, status, group, privacy,
   and search.
2. Admins create or update credentials, metadata, group bindings, proxy, model
   allow lists, rate multipliers, and RPM/base limits.
3. Admins can test an account against a model.
4. Admins can clear errors, clear rate-limit state, clear temporary
   unschedulable state, reset quota, refresh tier/usage, and mark schedulable.
5. Admins can import/export account data, batch create, bulk update, or batch
   refresh.

## OAuth onboarding and reauth

1. Admin generates an auth URL for Claude, OpenAI, Gemini, or Antigravity.
2. The user completes provider authorization.
3. Admin exchanges the code with session/state data.
4. The backend stores refresh/access token data in the account credentials.
5. Reauth or refresh later updates token fields while preserving other account
   credentials.

## Important rules

- Mixed-channel binding can return conflict behavior unless confirmed.
- OAuth-only groups reject API-key account types.
- Negative rate multipliers and invalid RPM/base values are rejected.
- OAuth code exchange requires session ID, code, and state.
- Some UI actions appear only for matching platform, account type, or status.

## Source map

- `backend/internal/server/routes/admin.go`
- `backend/internal/handler/admin/account_handler.go`
- `backend/internal/handler/admin/openai_oauth_handler.go`
- `backend/internal/handler/admin/gemini_oauth_handler.go`
- `backend/internal/handler/admin/antigravity_oauth_handler.go`
- `backend/internal/service/account_service.go`
- `backend/internal/service/oauth_service.go`
- `backend/internal/service/openai_oauth_service.go`
- `backend/internal/service/gemini_oauth_service.go`
- `backend/internal/service/antigravity_oauth_service.go`
- `frontend/src/views/admin/AccountsView.vue`
- `frontend/src/components/admin/account/*`
- `frontend/src/components/account/OAuthAuthorizationFlow.vue`

