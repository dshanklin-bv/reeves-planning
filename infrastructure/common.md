# reeves-common: Shared Infrastructure

`reeves-common` is a Python library that provides shared patterns for all reeves service modules. It is the foundation that makes services work behind the Caddy gateway and look consistent.

## What It Provides

### Web App Factory

```python
from reeves_common.web import make_app, template_context

app, templates = make_app(
    title="reeves-finance",
    module_static=here / "static",
    module_templates=here / "templates",
)
```

`make_app()` returns a FastAPI app + Jinja2 templates instance with:
- `X-Forwarded-Prefix` middleware (reads the header Caddy sets, applies base path)
- `/healthz` endpoint (liveness probe — "is the process alive?")
- `/readyz` endpoint (readiness probe — "is it ready for traffic?")
- Common static files mounted at `/static/common/`
- Service-specific static files mounted at `/static/`

### Template Context

```python
return templates.TemplateResponse("index.html", template_context(
    request,
    active_page="home",
    custom_var="value",
))
```

`template_context()` injects `base_path` into every template render. Templates then use:
```html
<link href="{{ base_path }}/static/common/base.css" rel="stylesheet">
<a href="{{ base_path }}/transactions">Transactions</a>
```

### Shared CSS

`/static/common/base.css` provides the dark theme used across all services:
- CSS custom properties for colors, spacing, typography
- Layout primitives (`.app`, `.sidebar`, `.content`, `.page`)
- Component styles (`.data-table`, `.stat-card`, `.badge`, `.sidebar-nav`)

Services link this in their base template and add service-specific CSS on top.

### Health Endpoints

Every service using `make_app()` gets:
- `GET /healthz` → `{"status": "healthy"}` (200)
- `GET /readyz` → `{"status": "ready"}` (200)
- `GET /version` → service name, version, Python version, uptime

The daemon probes these over the Unix domain socket to determine service state. Services can override `/readyz` with dependency checks (e.g., database connectivity).

## DB Patterns (planned)

- Soft delete mixins (`deleted_at` column, filtered queries)
- SQLAlchemy base classes for common patterns
- Migration helpers

## Package

```toml
# In service pyproject.toml
dependencies = ["reeves-common"]
```

Installed as an editable package from `~/repos-personal/reeves-common/`.
