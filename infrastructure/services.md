# Anatomy of a Reeves Service

Every reeves service follows the same pattern. Understanding this pattern is essential before modifying any service.

## The Pattern

A service is a **data domain** — finance, email, health, relationships, etc. Each domain:

1. **Ingests data** from external sources (bank APIs, IMAP, Apple Health, etc.)
2. **Stores it** in PostgreSQL under its own schema (`finance.*`, `email.*`, etc.)
3. **Serves a web UI** via FastAPI behind the Caddy gateway
4. **Exposes a CLI** for command-line operations
5. **May expose an MCP server** for Claude Code integration

## Standard Module Structure

```
reeves-<name>/
├── pyproject.toml              # Package definition, dependencies, CLI entry points
├── reeves.yaml                 # (future) Unit definitions for the daemon
├── CLAUDE.md                   # AI session instructions — READ THIS FIRST
├── reeves_<name>/              # Python package
│   ├── core.py                 # Domain logic (pure functions, no I/O framework)
│   ├── db.py                   # SQLAlchemy models + database operations
│   ├── cli.py                  # Click CLI commands
│   ├── mcp.py                  # MCP server for Claude Code (optional)
│   ├── ingest/                 # Data ingestion from external sources
│   └── ...                     # Domain-specific modules
├── web/
│   ├── app.py                  # FastAPI app using make_app() from reeves-common
│   ├── templates/              # Jinja2 templates using {{ base_path }}
│   └── static/                 # Service-specific CSS/JS
└── migrations/
    └── 001_initial.sql         # PostgreSQL schema under <name>.* namespace
```

## Database Convention

Each service owns a PostgreSQL schema matching its name:

- `finance.*` — accounts, transactions, journal entries
- `email.*` — accounts, threads, messages, labels
- `health.*` — imports, records, vitals, workouts, sleep
- `relationships.*` — persons, organizations, interactions, graph edges

The kernel (reeves-web) uses the `public` schema + a local SQLite for tasks/skillflows.

Bronze/silver/gold medallion pattern applies to raw data ingestion:
- `bronze.*` — raw synced data from Apple sources (via tosh)
- `silver.*` — cleaned, deduplicated
- `gold.*` — aggregated, insights

**Rule: No direct writes to gold tables.** Always go through ETL.

## How Services Run

All services are managed by `reeves-daemon`:

1. Service is defined in `~/.config/reeves/services.yaml`
2. Daemon generates a launchd plist and run script
3. `reeves-daemon install <name>` loads it into launchd
4. Service runs as uvicorn on a Unix domain socket (`~/.local/run/reeves/<name>.sock`)
5. Caddy routes `localhost:8787/<name>/` to the socket

**Do NOT run services manually, install global packages, or create standalone launchd plists.** Everything goes through the daemon.

## reeves-common Integration

Every service web app uses `reeves-common` for:

```python
from reeves_common.web import make_app, template_context

app, templates = make_app(
    title="reeves-finance",
    module_static=here / "static",
    module_templates=here / "templates",
)
```

This provides:
- Proxy-aware routing (`X-Forwarded-Prefix` handling)
- `/healthz` and `/readyz` endpoints (for daemon health probes)
- Shared CSS mount (`/static/common/base.css` — dark theme)
- `template_context()` for injecting `base_path` into templates

## Service Registry

Current services (as of Feb 2026):

| Service | Schema | Port | Status |
|---------|--------|------|--------|
| kernel (reeves-web) | public + SQLite | 8765 | Production |
| finance | finance.* | 8766 | Extracted |
| relationships | relationships.* | 8767 | Development |
| wealth | wealth.* | 8768 | Vision |
| tax | tax.* | 8769 | Vision |
| email | email.* | 8770 | Phase 1 |
| messages | messages.* | 8771 | Early |
| health | health.* | 8772 | Development |
| mechanic | (stateless) | 8773 | Infrastructure |
| router | (stateless) | 8774 | v1 — keyword matching |

## Adding a New Service

1. Create repo: `reeves-<name>/`
2. Follow the module structure above
3. Create PostgreSQL migration under `<name>.*` schema
4. Use `make_app()` from reeves-common for the web app
5. Add entry to `~/.config/reeves/services.yaml`
6. Run `reeves-daemon install <name>`
7. Add Caddy route via `reeves-daemon gateway reload`
8. Verify via `reeves-daemon status` and `localhost:8787/<name>/`
