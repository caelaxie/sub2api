# System settings and gateway behavior

Settings let admins configure registration, auth providers, SMTP, security,
branding, payment visibility, gateway scheduling behavior, cooldown policies,
stream timeout behavior, request rectification, beta policy, web-search
emulation, and admin API keys.

```text
admin settings page
  |
  +-- load settings
  +-- edit grouped settings
  +-- test SMTP or web search
  +-- save settings
  +-- services react through caches/callbacks
```

## Main flow

1. The settings page loads all admin settings.
2. Admins edit tabbed sections for auth, registration, providers, SMTP,
   security, UI, gateway, payment, and operational behavior.
3. SMTP can be tested before saving mail settings.
4. Admin API key can be read, regenerated, or deleted.
5. Gateway cooldown settings tune behavior for 529 overload and 429 rate-limit
   cases.
6. Stream timeout settings define how long streaming requests can stall.
7. Rectifier, beta policy, and web-search emulation rules change request
   handling at gateway time.
8. Settings updates can invalidate frontend HTML cache and refresh CSP
   frame-src origins.

## Important rules

- TOTP requires an encryption key in the environment.
- Enabled OAuth providers require client IDs, secrets, and valid callback URLs.
- SMTP tests require a host.
- Rectifier patterns have count and length limits.
- Some settings are read by background services and may reload schedules or
  runtime behavior.

## Source map

- `backend/internal/server/routes/admin.go`
- `backend/internal/handler/admin/setting_handler.go`
- `backend/internal/service/setting_service.go`
- `backend/internal/server/router.go`
- `frontend/src/views/admin/SettingsView.vue`

