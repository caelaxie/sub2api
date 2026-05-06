# Third-party OAuth and identity binding

LinuxDo, WeChat, and OIDC use the same broad pattern: start a browser OAuth
redirect, validate the provider callback, create a pending local session, then
let the frontend finish login, registration, or binding.

```text
browser
  |
  +-- /api/v1/auth/oauth/{provider}/start
  |      stores state/session cookies
  |
  +-- provider login
  |
  +-- /api/v1/auth/oauth/{provider}/callback
         validates callback and creates pending session
            |
            +-- existing identity -> login completion
            +-- new identity      -> create account or bind login
            +-- binding intent    -> bind to current user
```

## Main flow

1. The frontend redirects to the provider-specific start endpoint.
2. The backend stores state, redirect target, intent, browser-session cookies,
   and provider-specific data such as PKCE verifier or nonce.
3. The provider callback validates state and browser-session cookies.
4. The backend exchanges the OAuth code for provider tokens or user info.
5. The provider identity is normalized into a local identity key.
6. If that identity is already bound, the backend creates a pending login
   completion session.
7. If the identity is new, the backend creates a pending choice session. The
   user may need invitation validation, profile adoption, email verification,
   create-account, or bind-login.
8. The frontend calls `/auth/oauth/pending/exchange` to finish the pending
   state and receive tokens or the next required action.

## Binding from the profile page

1. The authenticated user asks to bind an identity provider.
2. The backend prepares an HttpOnly bind token.
3. The browser is redirected to the provider bind-start route.
4. The callback validates that the OAuth identity can be attached to the current
   user.
5. Conflicts are rejected instead of silently stealing another user's identity.

## Provider-specific notes

- OIDC can validate ID token, nonce, issuer, subject, matching userinfo subject,
  and verified email.
- WeChat may require `unionid`.
- LinuxDo and OIDC can find compatible existing users by email depending on the
  pending identity path.

## Important rules

- Missing callback parameters, invalid state, missing browser session, missing
  verifier, token exchange failure, and userinfo failure all stop the flow.
- Pending sessions can ask for invitation, email verification, TOTP, profile
  adoption, create-account, or bind-login.
- Provider mismatch in a pending operation is rejected.

## Source map

- `backend/internal/handler/auth_linuxdo_oauth.go`
- `backend/internal/handler/auth_oidc_oauth.go`
- `backend/internal/handler/auth_wechat_oauth.go`
- `backend/internal/handler/auth_oauth_pending_flow.go`
- `frontend/src/components/auth/LinuxDoOAuthSection.vue`
- `frontend/src/components/auth/OidcOAuthSection.vue`
- `frontend/src/components/auth/WechatOAuthSection.vue`
- `frontend/src/views/auth/LinuxDoCallbackView.vue`
- `frontend/src/views/auth/OidcCallbackView.vue`
- `frontend/src/views/auth/WechatCallbackView.vue`

