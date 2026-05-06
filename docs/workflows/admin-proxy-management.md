# Admin proxy management

Proxies are managed separately from accounts so multiple upstream accounts can
share a tested network route.

```text
admin proxy page
  |
  +-- create proxy
  +-- test connectivity
  +-- quality check
  +-- attach to accounts
```

## Main flow

1. Admins create proxies with protocol, host, port, and optional credentials.
2. Admins list and filter proxies.
3. The backend can test a proxy's connectivity and latency.
4. Quality checks probe more detailed network information.
5. Admins can see which accounts use a proxy.
6. Proxies can be batch created, batch deleted, imported, and exported.

## Important rules

- Supported protocols are `http`, `https`, `socks5`, and `socks5h`.
- Ports must be between 1 and 65535.
- Batch create skips duplicates by connection tuple.
- Account testing and OAuth flows can use selected proxies.

## Source map

- `backend/internal/server/routes/admin.go`
- `backend/internal/handler/admin/proxy_handler.go`
- `backend/internal/service/proxy_service.go`
- `backend/internal/repository/proxy_repo.go`
- `frontend/src/views/admin/ProxiesView.vue`
- `frontend/src/components/admin/proxy/ImportDataModal.vue`
- `frontend/src/components/common/ProxySelector.vue`
