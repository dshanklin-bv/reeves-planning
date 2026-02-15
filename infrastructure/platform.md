# Platform: Why Mac-Only

## The Thesis

The richest personal data is trapped in Apple's walled garden. Linux servers **cannot access**:

- `~/Library/Messages/chat.db` (iMessage/SMS)
- `~/Pictures/Photos Library.photoslibrary` (iCloud Photos)
- `~/Library/Application Support/Knowledge/knowledgeC.db` (Screen Time)
- `~/Library/Group Containers/.../NoteStore.sqlite` (Apple Notes)
- `~/Library/Safari/History.db` (Safari)
- Keychain, Call History, AddressBook

Mac-only isn't a limitation. The Mac IS the source. No other platform can access this data.

## Architecture Decision

| Factor | Server + Mac | Mac-Only |
|--------|-------------|----------|
| Privacy | Data in transit | Data never leaves home |
| Latency | Network hop | Local = instant |
| Complexity | Two systems | One system |
| Cost | Server + Mac | Mac only |
| Resilience | Depends on internet | Works offline |

## Hardware

**Development machine:** MacBook Pro M4 Pro 36GB (current — where everything runs today)
**Target home server:** Mac Mini M4 Pro 48GB (planned — always-on headless data appliance)

Both support:
- Multiple 7-13B models simultaneously (Ollama)
- Quantized 70B for complex tasks
- RAG with embeddings in memory
- Sub-second inference latency
- 9+ services running concurrently via launchd

## Data Flow

```
Apple Sources                    Local Postgres
---                              ---

Messages.db ──────┐
Photos      ──────┤
Notes       ──────┼───► tosh daemon ───► bronze.* tables
Screen Time ──────┤                          │
Browsers    ──────┤                          ▼
Calls       ──────┘                     silver.* tables
                                             │
                                             ▼
                                        gold.* tables
                                             │
                                             ▼
                                   Service modules query
                                   (finance, email, health, etc.)
```

## Network Access

Tailscale VPN connects all devices. No port forwarding, no public endpoints.

```
Mac (dev or home) ◄──── Tailscale ────► Other devices
     │
     └── Local network when home
```

## Backup

| Layer | Method | Frequency |
|-------|--------|-----------|
| Local | Time Machine | Hourly |
| Offsite | Backblaze B2 (~$5/mo) | Daily |

## What About rhea-dev?

Server is mothballed, not deleted. Can return if needed for public API endpoints, heavy GPU compute, or multi-user access. For now, Mac-only is simpler and more private.

## Implication for Portability

This is a macOS-native system by design. The daemon uses `launchd` because that's the Mac's process manager. The data sources are macOS-only databases. "Could this run on Linux" is the wrong question — the data doesn't exist on Linux.
