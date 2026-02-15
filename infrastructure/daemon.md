# The Daemon: reeves-daemon

The daemon is the control plane for the OBSERVE step of the loop. It manages the lifecycle of every process in the reeves ecosystem via macOS `launchd`.

## What It Does

1. Reads service definitions from `~/.config/reeves/services.yaml`
2. Generates launchd plists for each service
3. Runs services as uvicorn on Unix domain sockets
4. Monitors health via layered probes (socket → HTTP → launchd state)
5. Provides CLI for fleet management

## Service Lifecycle

```
install  →  generates plist + run script, loads into launchd
start    →  launchctl load (launchd starts the process)
stop     →  launchctl unload
restart  →  unload + load
status   →  layered health check (see State Machine below)
logs     →  tail service log files
```

## State Machine

Services are classified into states based on layered probes:

| State | How It's Determined |
|-------|-------------------|
| `online` | Socket connects + `/healthz` responds + `/readyz` responds |
| `degraded` | `/healthz` OK but `/readyz` failing (e.g., lost DB connection) |
| `hung` | PID exists, socket connects, but `/healthz` times out |
| `starting` | PID exists but socket not yet connectable |
| `crashed` | No PID, non-zero last exit code |
| `crash_loop` | Crashed + high run count (repeated restarts) |
| `error` | Loaded in launchd but no PID and clean exit (config issue) |
| `offline` | Not loaded in launchd |

Probe order: launchctl state → socket connectivity → HTTP `/healthz` over UDS → HTTP `/readyz` over UDS → `launchctl print` for exit codes.

## Key Files

| File | Purpose |
|------|---------|
| `~/.config/reeves/services.yaml` | Service registry (names, repos, ports, labels) |
| `~/.config/reeves/Caddyfile` | Generated gateway config |
| `~/.local/run/reeves/*.sock` | Unix domain sockets per service |
| `~/Library/LaunchAgents/com.reeves.*.plist` | Generated launchd plists |
| `~/.local/share/reeves/logs/` | Service log files |

## Module Structure

```
reeves_daemon/
├── config.py        # Load services.yaml, DaemonConfig, ServiceConfig
├── manager.py       # Orchestration: install/start/stop/status + state machine
├── launchd.py       # Plist generation, launchctl operations, launchd info parsing
├── probes.py        # Socket checks, HTTP health probes over UDS
├── caddy.py         # Caddyfile generation, reload, route verification via Admin API
└── cli.py           # Click CLI: status, start, stop, install, gateway commands
```

## The Multi-Process Future

Today: one service = one web process. The architecture is designed to evolve toward multiple process types per service — jobs, daemons, and socket-activated processes — all managed through launchd.

See **[processes.md](processes.md)** for the full design: unit kinds, config schema, trigger mapping, runtime knobs, and implementation phases.

## CLI Reference

```
reeves-daemon status              # Fleet overview: service, status, PID, health, ready, latency
reeves-daemon status --json       # Machine-readable output
reeves-daemon start <name>        # Start a service
reeves-daemon stop <name>         # Stop a service
reeves-daemon restart <name>      # Restart a service
reeves-daemon install <name>      # Generate plist + run script, load into launchd
reeves-daemon logs <name>         # Tail service logs
reeves-daemon gateway status      # Caddy port check + route verification via Admin API
reeves-daemon gateway reload      # Regenerate Caddyfile and reload Caddy
```
