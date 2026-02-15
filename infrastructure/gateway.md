# Gateway: Caddy Reverse Proxy

All services are accessed through a single gateway — Caddy reverse proxy on **port 8787** — with path-based routing to Unix domain sockets.

## Why This Architecture

- **Single entry point**: `localhost:8787` routes to all services
- **Unix domain sockets**: No port conflicts, no TCP overhead, better security
- **Path-based routing**: `/finance/` → finance socket, `/email/` → email socket
- **Kernel as root**: `/` routes to reeves-web (the kernel/OS)

## Route Layout

```
localhost:8787/              →  kernel (reeves-web)
localhost:8787/finance/      →  ~/.local/run/reeves/finance.sock
localhost:8787/email/        →  ~/.local/run/reeves/email.sock
localhost:8787/health/       →  ~/.local/run/reeves/health.sock
localhost:8787/relationships/→  ~/.local/run/reeves/relationships.sock
localhost:8787/wealth/       →  ~/.local/run/reeves/wealth.sock
localhost:8787/tax/          →  ~/.local/run/reeves/tax.sock
localhost:8787/messages/     →  ~/.local/run/reeves/messages.sock
localhost:8787/mechanic/     →  ~/.local/run/reeves/mechanic.sock
```

## How It Works

Each non-kernel route in the Caddyfile:

```
handle /finance/* {
    uri strip_prefix /finance
    reverse_proxy unix//Users/dshanklinbv/.local/run/reeves/finance.sock {
        header_up X-Forwarded-Prefix /finance
    }
}
```

1. Caddy matches the path prefix
2. Strips the prefix before forwarding
3. Sets `X-Forwarded-Prefix` header so the app knows its base path
4. Forwards to the Unix domain socket

The kernel route is the fallback — anything that doesn't match a service prefix goes to kernel.

## Proxy-Aware Services

Services must be proxy-aware to work behind Caddy. `reeves-common` provides this via `make_app()`:

- `make_app()` reads `X-Forwarded-Prefix` and sets `base_path` for templates
- `template_context()` injects `base_path` into every template render
- Templates use `{{ base_path }}` to prefix all URLs
- `get_base_path(request)` used in redirect responses

See `common.md` for the full pattern.

## Caddy Admin API

Caddy exposes an admin API at `localhost:2019` for route verification:

```
GET http://localhost:2019/config/    # Full Caddy config as JSON
```

The daemon's `gateway status` command queries this to verify each service has a route pointing to its socket.

## Key Files

| File | Purpose |
|------|---------|
| `~/.config/reeves/Caddyfile` | Generated Caddy configuration |
| `~/.local/run/reeves/*.sock` | Unix domain sockets per service |

## Troubleshooting

- **502 Bad Gateway**: Socket exists but service isn't running. Check `reeves-daemon status`.
- **404**: Route exists but service returned 404. Service is running but may not have `make_app()`.
- **Connection refused on 8787**: Caddy isn't running. `brew services restart caddy`.
