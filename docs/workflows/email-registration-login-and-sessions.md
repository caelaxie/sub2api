# Email registration, login, and sessions

This workflow covers normal email signup, email/password login, optional TOTP
login, token refresh, logout, and password reset.

```text
visitor
  |
  +-- public settings decide available auth features
  |
  +-- register or login
        |
        +-- email verification required? send code first
        +-- TOTP enabled for login? return temp token, then verify TOTP
        +-- success: return access token and refresh token
```

## Registration

1. The frontend loads public settings to decide whether registration, email
   verification, promo codes, invitation codes, OAuth buttons, Turnstile, and
   email suffix limits are enabled.
2. The frontend can pre-validate promo and invitation codes.
3. If email verification is enabled, the registration form is stored in browser
   session storage, a verification code is sent, and the final register call
   includes `verify_code`.
4. If email verification is disabled, the register call is sent directly.
5. The backend validates settings, user input, Turnstile if enabled, codes, and
   any affiliate or promo data before creating the user.

## Login and sessions

1. The user submits email, password, and Turnstile token when required.
2. The backend checks credentials, backend mode, user status, and TOTP policy.
3. If TOTP is required, the login response includes `requires_2fa` and a
   temporary token. The frontend then calls the TOTP login endpoint.
4. A successful login returns access and refresh tokens.
5. Refresh rotates the token pair.
6. Logout revokes the refresh token when one is present.
7. Revoke-all-sessions invalidates all current sessions for the authenticated
   user.

## Password reset

1. The user asks for a password reset email.
2. The backend sends a reset link if reset email is configured, but does not
   reveal whether the email exists.
3. The reset screen reads the `email` and `token` query parameters and submits
   the new password.

## Important rules

- Risky public auth endpoints are rate-limited and fail closed when the rate
  limiter cannot run.
- Backend mode can block non-admin login and refresh.
- Missing frontend URL makes password reset unavailable.
- Invalid or expired TOTP temp tokens are rejected.
- Invitation and promo validation return specific error codes so the UI can show
  clearer messages.

## Source map

- `backend/internal/server/routes/auth.go`
- `backend/internal/handler/auth_handler.go`
- `frontend/src/views/auth/RegisterView.vue`
- `frontend/src/views/auth/EmailVerifyView.vue`
- `frontend/src/views/auth/LoginView.vue`
- `frontend/src/views/auth/ForgotPasswordView.vue`
- `frontend/src/views/auth/ResetPasswordView.vue`
- `frontend/src/api/auth.ts`

