# reeves-mechanic: Fleet Health & Compliance

Mechanic is the integration testing and monitoring tool for the reeves ecosystem. It verifies that services work end-to-end through the Caddy gateway and tracks fleet-wide compliance with architectural standards.

## What It Does

1. **Health checks** — HTTP requests through the gateway to each service
2. **Gateway checks** — Caddy port, socket existence, route resolution, trailing slash normalization
3. **Smoke tests** — Playwright launches Chromium, navigates every page, captures screenshots, checks CSS loaded, collects console errors
4. **Fleet-wide scanners** — Audit all service repos for compliance with patterns (proxy migration, shared CSS, health endpoints)

## CLI

```
mechanic status               # Fleet overview table
mechanic health               # Fast HTTP liveness checks through the gateway
mechanic gateway              # Caddy port, sockets, routes, conflicts
mechanic smoke                # Full Playwright suite with screenshots
mechanic smoke --no-browser   # Health + gateway only (fast)
mechanic smoke --service X    # Target a single service
mechanic health --json        # Machine-readable output
```

## Scanners

Scanners audit every service repo for compliance with a specific concern:

### Proxy Migration
Checks each service for: `make_app()`, `template_context()`, `{{ base_path }}` in templates, `get_base_path()` in redirects, `/healthz` endpoint.

### Common CSS Adoption
Checks each service for: `base.css` linked in templates, `mount_common_static()` called.

### Health Monitoring Readiness
Probes each running service for: socket connectable, `/healthz` returns 200, `/readyz` returns 200.

## Web Dashboard

Mechanic runs as a service at `localhost:8787/mechanic/` with:
- Fleet status table (service, status, socket, health, ready, latency, pages)
- Scanner results with per-service compliance breakdowns
- Latest smoke test run results

## Reports

Smoke test runs are saved to `~/.reeves/mechanic/runs/<timestamp>/` with:
- `run.json` — structured results
- `report.html` — dark theme HTML report with embedded screenshots

## Key Distinction

The **daemon** handles direct health probes (UDS → `/healthz`). **Mechanic** handles gateway integration tests (browser → Caddy → UDS → response). They're complementary — daemon tells you if a service is alive, mechanic tells you if a user can reach it.
