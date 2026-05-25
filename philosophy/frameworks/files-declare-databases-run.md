# Files Declare, Databases Run

## The Principle

In every production system that manages processes — from Kubernetes to systemd to launchd — the same split appears: **static files are the human source of truth, and a dynamic database is the machine source of truth.**

Files declare what should exist. Databases track what does exist.

This is not a convention. It is a convergent solution discovered independently by every system that has to manage processes at scale.

## The Evidence

| System | Declaration (files) | Runtime State (database) |
|--------|-------------------|--------------------------|
| Kubernetes | YAML manifests in git | etcd (distributed key-value store) |
| systemd | Unit files on disk (.service, .timer) | In-memory dependency graph |
| Docker Compose | docker-compose.yml | Swarm's internal Raft DB |
| Heroku | Procfile in repo root | Platform database |
| HashiCorp Nomad/Consul | HCL job files | Consul service registry |
| AWS ECS | Task Definition JSON / CloudFormation | AWS control plane |
| macOS launchd | .plist files in LaunchAgents | launchd internal state |
| macOS XPC | Info.plist in app bundle | launchd on-demand activation |

Zero exceptions. No production system stores "what processes should exist" in the application database.

## Why This Split Exists

### The Bootstrap Problem

A process manager must know what to start before anything else is running. If the config lives in PostgreSQL, the manager can't read its config until PostgreSQL is up. But it can't start PostgreSQL without reading the config. Files on disk are always available — they survive reboots, crashes, and database corruption.

### Version Control

Files are diffable, reviewable, and auditable. `git log` tells you who changed what and when. Database rows require a separate audit system to achieve the same thing.

### Failure Isolation

If an application's database is corrupted or a migration is broken, the process manager can still read the declaration files and restart the service. If the declaration is in the broken database, the process manager is blind.

### Separation of Concerns

The application database stores application data (messages, transactions, contacts). The process declaration is infrastructure — it describes how the application runs, not what the application does. Mixing these creates a category error.

## How This Applies to Reeves

### The App/Service Model

Each reeves module (email, messages, health, finance) is an **app** that can declare multiple **units** — child processes the daemon manages:

- **Web units** — uvicorn on Unix socket behind Caddy (always-on)
- **Job units** — run-to-completion tasks on a schedule or file trigger (sync jobs)
- **Daemon units** — long-running non-web processes (MCP servers, queue workers)

### The Correct Architecture

```
DECLARATION (files, version-controlled)
├── reeves.yaml in each app repo     ← "I need a web server and a sync job"
└── ~/.config/reeves/services.yaml   ← machine-local overrides (ports, env, disabled units)

CONTROL PLANE (reeves-daemon)
├── Reads declarations at install time
├── Generates launchd plists per unit
├── Manages lifecycle (start/stop/restart)
└── Runs health probes

RUNTIME STATE (database)
├── Job execution history (SQLite: jobs.db)
├── Last sync timestamps
├── Health status snapshots
└── Exit codes and error logs
```

### The Merge Rule

When both `reeves.yaml` (in the repo) and `services.yaml` (on the machine) define the same unit, the machine-local config wins. This handles:

- Local differences (laptop vs desktop)
- Temporarily disabling a noisy job
- Environment-specific secrets or ports
- Development overrides

### What Does NOT Go in the Database

- What services exist
- What processes each service needs
- How those processes should be triggered (schedule, file watch, socket)
- What command to run

### What DOES Go in the Database

- When the last sync ran
- Whether it succeeded or failed
- How long it took
- What errors occurred
- Current health state

## The Single Tunnel Corollary

This principle reinforces the Single Tunnel rule: all processes flow through one governed path (the daemon). The daemon is the only component with authority to install, start, stop, and monitor processes. If an app could self-register processes by writing to a database, it would be building a second airport — bypassing the daemon's governance, visibility, and lifecycle management.

Files submitted to the daemon are like flight plans submitted to air traffic control. The control tower decides when and how planes take off. Planes don't self-authorize.

## The Biological Analog

DNA is a file — a static, declarative specification that describes what an organism needs. The cellular machinery reads the file and executes it. The organism's runtime state (metabolite levels, gene expression, protein concentrations) is the "database" — dynamic, queryable, responsive to environment.

No organism stores its own genome in its metabolic state. The declaration is separate from the execution. The spec is durable. The state is ephemeral.

This is the same convergent solution, discovered by biology and by every process management system humans have built.
