# Multi-Process Architecture

Today, each reeves service is a single web process. The daemon manages 9 uvicorn instances on Unix domain sockets behind Caddy. This document describes the architecture for extending the daemon to manage multiple process types per service.

## Why

The daemon is the control plane for the OBSERVE step of the loop. Observation requires more than web servers:

- **Scheduled jobs** — fetch Gmail every 15 minutes, sync Monarch Money daily, run tosh extraction hourly
- **File-triggered jobs** — re-process Messages.db when macOS updates it, re-index photos when library changes
- **On-demand daemons** — MCP servers that spin up on first connection and exit after idle timeout
- **Always-on daemons** — queue workers, embedding generators, voice bridge

Each of these maps to a launchd primitive. The daemon already speaks launchd. It just needs to generate more than one plist per service.

## Unit Kinds

Every process managed by the daemon is a **unit**. Three kinds:

| Kind | Purpose | launchd Mechanism | Lifecycle |
|------|---------|-------------------|-----------|
| `web` | uvicorn on UDS behind Caddy | `KeepAlive: true` | Always running, auto-restart on crash |
| `job` | Run-to-completion task | `StartCalendarInterval` or `WatchPaths` | Starts on trigger, exits when done |
| `daemon` | Long-running non-web process | `Sockets` or `KeepAlive` | Socket-activated or always-on |

### Web Units
What exists today. uvicorn serves a FastAPI app on a Unix domain socket. Caddy routes HTTP traffic to it. KeepAlive ensures launchd restarts it if it crashes.

### Job Units
New. A Python script or CLI command that runs to completion. launchd launches it on a schedule (`StartCalendarInterval`) or when a file changes (`WatchPaths`). It does its work and exits. launchd tracks the exit code.

Examples:
- Email fetch: runs every 15 minutes, syncs new messages from Gmail
- Tosh extraction: runs when `~/Library/Messages/chat.db` changes
- Monarch sync: runs daily at 6am, pulls transaction data

### Daemon Units
New. A long-running process that isn't a web server. Two flavors:

**Socket-activated:** launchd creates and owns the socket. On first connection, launchd starts the process and hands it the socket. After idle timeout, the process exits. Scale-to-zero for MCP servers that are rarely used.

**Always-on:** like web, but not behind Caddy. For processes that need to be continuously running (voice bridge, embedding worker).

## Configuration

### Shorthand (80% of services)

Most services are web-only. The existing config works unchanged:

```yaml
finance:
  label: com.reeves.finance
  repo: reeves-finance
  app: web.app:app
  port: 8766
```

When `app:` is present and no `units:` block exists, the daemon generates a single web unit with KeepAlive.

### Explicit Units (complex services)

Services with multiple processes declare them in `units:`:

```yaml
email:
  label: com.reeves.email
  repo: reeves-email
  app: web.app:app           # still has a web UI
  port: 8770
  units:
    fetch:
      kind: job
      exec: ["python", "-m", "email_service.jobs.fetch"]
      triggers:
        - schedule: { Minute: 0, Hour: "*/1" }
      runtime:
        timeout_seconds: 300
        concurrency: forbid
    mcp:
      kind: daemon
      exec: ["python", "-m", "email_service.mcp_server"]
      triggers:
        - socket: { name: mcp, idle_exit_seconds: 600 }

messages:
  label: com.reeves.messages
  repo: reeves-messages
  app: web.app:app
  port: 8771
  units:
    tosh-sync:
      kind: job
      exec: ["python", "-m", "reeves_messages.jobs.tosh_sync"]
      triggers:
        - watch_paths: ["~/Library/Messages/chat.db"]
      runtime:
        timeout_seconds: 120
        concurrency: forbid
```

The web unit is always implicit when `app:` is present. You don't need to declare it in `units:`.

## Triggers

Triggers are the "when" for a unit. They map directly to launchd plist keys:

| Trigger | launchd Key | Example |
|---------|------------|---------|
| `schedule` | `StartCalendarInterval` | `{ Minute: 0, Hour: "*/1" }` — every hour on the hour |
| `watch_paths` | `WatchPaths` | `["~/Library/Messages/chat.db"]` — when file changes |
| `socket` | `Sockets` | `{ name: mcp, idle_exit_seconds: 600 }` — on first connection |
| (none / always) | `KeepAlive: true` | Web units and always-on daemons |

A unit can have multiple triggers (e.g., a job that runs on schedule AND when a file changes).

### Schedule Format

Follows launchd's `StartCalendarInterval` keys:

```yaml
schedule:
  Minute: 0          # 0-59
  Hour: 6            # 0-23
  Day: 1             # 1-31
  Weekday: 1         # 0=Sunday, 1=Monday, ...
  Month: 1           # 1-12
```

Omitted keys mean "every" (like cron `*`).

## Runtime Knobs

Control how a unit runs:

| Key | Values | Default | Purpose |
|-----|--------|---------|---------|
| `timeout_seconds` | int | 0 (no limit) | Kill the process after this many seconds |
| `concurrency` | `allow`, `forbid`, `queue` | `allow` | What to do if triggered while already running |
| `retry_on_failure` | bool | false | Restart on non-zero exit (jobs only) |
| `env` | dict | {} | Extra environment variables |
| `working_dir` | path | repo root | Working directory for the process |

**Concurrency modes:**
- `allow` — multiple instances can run simultaneously
- `forbid` — skip the trigger if already running (launchd default for CalendarInterval)
- `queue` — wait for current run to finish, then start again

## Plist Generation

The daemon's launchd renderer generates one plist per unit:

```
com.reeves.email.plist          ← web unit (implicit)
com.reeves.email.fetch.plist    ← fetch job unit
com.reeves.email.mcp.plist      ← mcp daemon unit
```

Naming: `{service_label}.{unit_name}.plist`. The bare `{service_label}.plist` is always the web unit.

## Job Execution History

SQLite database at `~/.local/share/reeves/jobs.db` tracks every job run:

```sql
CREATE TABLE job_runs (
    id INTEGER PRIMARY KEY,
    service TEXT NOT NULL,
    unit TEXT NOT NULL,
    started_at TEXT NOT NULL,
    finished_at TEXT,
    exit_code INTEGER,
    duration_seconds REAL,
    trigger_type TEXT,       -- 'schedule', 'watch_paths', 'manual'
    stdout_tail TEXT,        -- last 1000 chars of stdout
    stderr_tail TEXT         -- last 1000 chars of stderr
);
```

This gives the daemon visibility into job health without parsing launchd logs.

## Ad-Hoc Triggers

Any job can be triggered manually:

```bash
launchctl kickstart -k gui/501/com.reeves.email.fetch
```

The `-k` flag kills a running instance first (respects `concurrency: forbid`). The daemon CLI wraps this:

```bash
reeves-daemon run email/fetch        # kick off the fetch job now
reeves-daemon history email/fetch    # show recent runs
```

## Pipeline Jobs

Some jobs depend on others. A tosh extraction must finish before downstream enrichment runs. Use `after:` to declare ordering:

```yaml
units:
  extract:
    kind: job
    exec: ["python", "-m", "reeves_messages.jobs.extract"]
    triggers:
      - watch_paths: ["~/Library/Messages/chat.db"]
  enrich:
    kind: job
    exec: ["python", "-m", "reeves_messages.jobs.enrich"]
    after: [extract]
```

This is NOT a DAG engine. It's simple sequential ordering. The daemon runs `extract`, waits for exit 0, then runs `enrich`. If `extract` fails, `enrich` doesn't run.

## Implementation Approach

**Spec-first.** The portable layer is OS-neutral domain objects:

```python
# spec.py — domain objects, no platform code
@dataclass
class TriggerSpec:
    schedule: dict | None = None
    watch_paths: list[str] | None = None
    socket: dict | None = None

@dataclass
class RuntimeSpec:
    timeout_seconds: int = 0
    concurrency: str = "allow"
    retry_on_failure: bool = False
    env: dict = field(default_factory=dict)

@dataclass
class UnitSpec:
    name: str
    kind: str                        # web, job, daemon
    exec: list[str] | None = None   # None for web (uvicorn)
    triggers: list[TriggerSpec] = field(default_factory=list)
    runtime: RuntimeSpec = field(default_factory=RuntimeSpec)
    after: list[str] = field(default_factory=list)
```

**Renderer.** The launchd-specific code turns specs into plists. One renderer, not a `backends/` directory with a protocol. If systemd support is ever needed, add a second renderer then — not now.

**No Docker.** The entire point of running on macOS is direct access to Apple data sources. Docker would cut off that access.

## Implementation Phases

These are future work. This document captures the design; implementation happens when a service actually needs multi-process support.

1. **Phase 1: Spec + Config** — `spec.py` domain objects, config parser that reads `units:` blocks from services.yaml
2. **Phase 2: Plist Renderer** — extend launchd.py to generate job/daemon plists from UnitSpec
3. **Phase 3: Manager** — extend manager.py to install/start/stop/status multi-unit services, run pipelines
4. **Phase 4: History** — SQLite job_runs table, CLI reporting (`reeves-daemon history`)

The trigger for Phase 1 is when email needs its fetch job managed by the daemon instead of a standalone launchd plist.
