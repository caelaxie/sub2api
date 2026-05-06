# Profile, TOTP, and notification email

This workflow is the authenticated user's account-maintenance area. It covers
profile edits, password changes, identity binding, TOTP setup/disable, and
balance notification email settings.

```text
authenticated user
  |
  +-- profile page
        |
        +-- update name/avatar
        +-- change password
        +-- bind or unbind identities
        +-- enable or disable TOTP
        +-- configure notification email
```

## Profile and password

1. The profile page refreshes the current user and public settings.
2. Username and avatar changes call the authenticated user update endpoint.
3. Password changes require the current password and new password.

## Identity binding

1. Email binding sends a verification code.
2. The user submits email, code, and password.
3. OAuth binding starts through the pending OAuth binding flow.
4. Unbinding removes the selected provider identity.
5. When an identity is actually removed, active user tokens are revoked so old
   sessions cannot keep using removed login methods.

## TOTP

1. The UI reads TOTP status.
2. Setup asks the backend which verification method applies: email or password.
3. After verification, the backend returns a secret, QR information, and setup
   token.
4. The user submits a six-digit TOTP code with the setup token.
5. Disable uses a similar verification step before removing TOTP.

## Notification email

1. The user sends a code to the notification email.
2. The user verifies the code.
3. The user can toggle notifications or remove the notification email.

## Important rules

- All operations require a valid JWT subject.
- TOTP can be globally disabled.
- TOTP enable requires exactly six digits and the setup token.
- Notification email verification also expects a six-digit code.
- Email identity binding requires a valid email, code, and password.

## Source map

- `backend/internal/server/routes/user.go`
- `backend/internal/handler/user_handler.go`
- `backend/internal/handler/totp_handler.go`
- `frontend/src/views/user/ProfileView.vue`
- `frontend/src/components/user/profile/ProfileEditForm.vue`
- `frontend/src/components/user/profile/ProfilePasswordForm.vue`
- `frontend/src/components/user/profile/ProfileIdentityBindingsSection.vue`
- `frontend/src/components/user/profile/ProfileTotpCard.vue`
- `frontend/src/components/user/profile/ProfileBalanceNotifyCard.vue`
- `frontend/src/components/user/profile/TotpSetupModal.vue`
- `frontend/src/components/user/profile/TotpDisableDialog.vue`

