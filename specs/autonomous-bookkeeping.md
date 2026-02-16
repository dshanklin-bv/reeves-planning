# Spec: Autonomous Bookkeeping

**Goal:** AI does Dallas's books. Transactions flow in from banks, get categorized, turned into journal entries, validated by Ledger CLI, and either auto-approved or queued for human review. The human touches the keyboard only for ambiguous items and monthly review.

**Status:** Draft
**Service:** reeves-finance
**Depends on:** reeves-daemon (scheduling), reeves-router + reeves-email (Increment 2)

---

## Current State (what exists)

| Component | Status | Notes |
|-----------|--------|-------|
| Bank ingestion (Tiller) | Built | `sync tiller` pulls from Google Sheets |
| Bank ingestion (Mercury) | Built | `sync mercury` pulls via API |
| CSV import (Chase/Amex) | Built | `import csv --bank chase\|amex` |
| Rule-based categorization | Built | 25 rules, regex on payee/description |
| Pattern analyzer | Built | `analyze` groups by payee, suggests accounts via ACCOUNT_HINTS |
| Rule learner | Built | `learn --apply` creates rules from analysis |
| Entry builder | Built | Builds balanced double-entry postings from transactions |
| Ledger CLI validation | Built | All entries pass `ledger balance` before storage |
| Auto-process pipeline | Built | `auto` chains analyze → learn → process |
| Approval workflow | Built | pending → approved/rejected, manual via CLI or web |
| Web dashboard | Built | Transaction queue, journal, entities, autopilot pages |
| Scheduled execution | **Missing** | `auto` must be run manually |
| Email receipt enrichment | **Missing** | No cross-service communication |
| LLM categorization | **Missing** | Analyzer uses regex only, no `claude -p` |
| Daily summary/notification | **Missing** | No report of what was auto-done |
| Confidence escalation | **Missing** | Low-confidence items just sit in pending |

**Live data:** 3,113 raw transactions, 110 processed, 3,003 unprocessed, 25 rules, all from Tiller.

---

## Confidence Policy

Confidence is a 0.0–1.0 score indicating how certain the system is about a transaction's account categorization. Multiple methods produce confidence, and they **do not combine** — each method either upgrades or replaces the previous confidence.

### Confidence Sources (in evaluation order)
1. **Rule match** → confidence from the matching `categorization_rules` row (typically 0.7–1.0)
2. **ACCOUNT_HINTS regex** → 0.7 (hardcoded in analyzer.py)
3. **Email enrichment** → if receipt found and matches, override confidence to 0.85
4. **LLM categorization** → LLM returns its own 0.0–1.0 confidence; replaces prior value

### Named Thresholds
| Threshold | Value | Purpose |
|-----------|-------|---------|
| `AUTO_APPROVE_THRESHOLD` | 0.8 | Below this → pending review. Always. |
| `ENRICH_TRIGGER_THRESHOLD` | 0.8 | Below this → attempt email enrichment (cheap, bounded) |
| `LLM_TRIGGER_THRESHOLD` | 0.5 | Below this → attempt LLM categorization (expensive, capped) |
| `LEARN_RULE_MIN_CONFIDENCE` | 0.5 | Minimum confidence to auto-create a rule from analysis |

### Policy
- If rule confidence >= 0.8 → auto-approve. No enrichment, no LLM.
- If rule confidence 0.5–0.79 → attempt enrichment. If enrichment boosts to >= 0.8, auto-approve.
- If rule confidence < 0.5 → attempt enrichment, then LLM if still below 0.5.
- If no rule matches (confidence 0.0) → create entry to `Expenses:Uncategorized`, mark pending.
- **Every transaction gets an entry.** Zero matches → Uncategorized/pending. Never dropped.

### High-Risk Categories
- Transfers between own accounts (`Transfers:*`) → never auto-approve unless confidence == 1.0 (explicit rule match)
- Amounts > $1,000 → require confidence >= 0.9 for auto-approve

---

## Prerequisites

The pipeline requires these to be present. Missing prerequisites cause that step to fail gracefully (logged, not fatal).

| Prerequisite | Required for | Failure behavior |
|--------------|-------------|-----------------|
| `~/.config/reeves-finance/tiller-service-account.json` | Tiller sync | Sync step skipped, logged |
| `MERCURY_API_TOKEN` env var | Mercury sync | Sync step skipped, logged |
| `ledger` CLI in PATH | Entry validation | Process step fails (entries can't be created) |
| PostgreSQL connectivity (`REEVES_DATABASE_URL`) | Everything | Pipeline aborts immediately |
| `~/.local/run/reeves/` directory | Lock file | Created automatically |

---

## Exit Code Policy

Pipeline exit codes for daemon job tracking:
- **Exit 0:** Pipeline completed. All steps ran (some may have had errors). Check JSON `ok` field.
- **Exit 1:** Pipeline could not run at all (lock held, DB unreachable, fatal error).

Partial failures (e.g., Tiller sync fails but processing succeeds) are **exit 0 with `ok: false`** in JSON. The daemon treats exit 0 as "job ran" and logs the JSON result. The `errors` array in the result shows what went wrong.

---

## Increment 1: Scheduled Pipeline

**What:** The daemon runs the finance auto-processing pipeline on a schedule. No human triggers it.

### Scenario 1.1: Nightly Sync + Process

```
GIVEN the daemon is running and finance is registered
WHEN the nightly schedule fires (e.g., 2:00 AM)
THEN the system:
  1. Runs `reeves-finance sync tiller --days 3`
  2. Runs `reeves-finance sync mercury --days 3`
  3. Runs `reeves-finance auto --confidence 0.8 --limit 200 --json`
  4. Logs the result to finance service log
  5. Exits cleanly with status in structured JSON
```

**Acceptance criteria:**
- [ ] New `reeves-finance run-pipeline` CLI command that chains sync → auto
- [ ] Pipeline outputs structured JSON: `{synced: {tiller: N, mercury: N}, processed: N, auto_approved: N, pending_review: N, failed: N}`
- [ ] Pipeline is idempotent: running twice produces no duplicates (external_id dedup)
- [ ] Pipeline completes in < 60 seconds for 200 transactions
- [ ] Errors in one step don't abort the whole pipeline (sync failure doesn't skip process)

### Scenario 1.2: Daemon Scheduled Job

```
GIVEN reeves-daemon has a job scheduler
AND finance has a registered pipeline job
WHEN the cron expression matches (0 2 * * *)
THEN the daemon executes `reeves-finance run-pipeline --json`
AND logs stdout/stderr to ~/.local/log/reeves-finance-pipeline.log
AND records the run result in daemon state
```

**Acceptance criteria:**
- [ ] `services.yaml` supports a `jobs` field per service (cron expression + command)
- [ ] Daemon spawns job as subprocess, captures output, tracks exit code
- [ ] `reeves-daemon jobs` CLI shows job history (last run, next run, status)
- [ ] Failed jobs don't crash the daemon; they're logged and retried next cycle

### Scenario 1.3: No-Match Transaction (Uncategorized)

```
GIVEN a raw transaction: {payee: "XYZ UNKNOWN VENDOR", amount: -42.00, date: "2025-02-08"}
AND no categorization rule matches (confidence 0.0)
AND no ACCOUNT_HINTS pattern matches
WHEN auto-process runs
THEN the entry builder creates a journal entry:
  debit: Expenses:Uncategorized $42.00
  credit: Assets:Bank:Mercury $-42.00
AND the entry is created with status "pending" (confidence 0.0 < 0.8)
AND the transaction is marked as processed (journal_entry_id set)
```

**Acceptance criteria:**
- [ ] Transactions with zero confidence still get entries (never dropped)
- [ ] Target account defaults to `Expenses:Uncategorized` (expense) or `Income:Other` (income)
- [ ] Entry status is "pending" (never auto-approved at confidence < 0.8)
- [ ] Transaction is linked to the entry via `journal_entry_id`

### Scenario 1.4: Pipeline Status Check

```
GIVEN the pipeline ran last night
WHEN a user or agent runs `reeves-finance status --json`
THEN the output includes:
  - last_pipeline_run: ISO timestamp
  - last_pipeline_result: {synced, processed, auto_approved, pending_review, failed}
  - unprocessed_count: N
  - pending_review_count: N
  - oldest_unprocessed_date: ISO date
```

**Acceptance criteria:**
- [ ] `status` command includes pipeline history
- [ ] Pipeline writes its last result to a known location (DB or file)

### Commands (Increment 1)

```bash
# New
reeves-finance run-pipeline [--days 3] [--confidence 0.8] [--limit 200] [--json]

# Modified
reeves-finance status --json  # Now includes pipeline stats

# New daemon capability
reeves-daemon jobs                    # List all scheduled jobs
reeves-daemon jobs run finance        # Manually trigger a job
reeves-daemon jobs history finance    # Show last N runs
```

### Boundaries (Increment 1)
- Pipeline MUST NOT approve entries with confidence < 0.8 (configurable threshold)
- Pipeline MUST NOT delete or modify existing entries
- Pipeline MUST NOT run concurrent instances (file lock or PID check)
- Sync failures MUST be logged but MUST NOT prevent processing of already-synced transactions
- All DB writes use soft deletes, session injection, Ledger CLI validation

---

## Increment 2: Email Receipt Enrichment

**What:** When categorizing a transaction, finance asks the router for help. The router directs it to email, which searches for matching receipts. The receipt context improves categorization confidence.

### Scenario 2.1: Receipt Lookup for Ambiguous Transaction

```
GIVEN a raw transaction: {payee: "HILTON", amount: -450.00, date: "2025-02-03"}
AND the categorizer returns confidence < 0.8 (below auto-approve threshold)
WHEN the pipeline processes this transaction
THEN finance calls the router: "I need context for a $450 HILTON charge on Feb 3"
AND the router returns: [{service: "email", capability: "get_receipt", params: {merchant: "Hilton", date: "2025-02-03", amount: 450.00}}]
AND finance calls email's receipt endpoint
AND email returns: {found: true, subject: "Hilton Garden Inn - Booking Confirmation", from: "no-reply@hilton.com", body_snippet: "Business trip - Indianapolis conference"}
AND finance uses the receipt context to categorize as Expenses:Travel:Hotels with confidence 0.85
AND the entry is auto-approved (confidence >= 0.8)
```

**Acceptance criteria:**
- [ ] `entry_builder.py` has an `enrich_transaction()` step before categorization
- [ ] Enrichment calls router via HTTP (not direct import — services are decoupled)
- [ ] Router query returns structured capability matches
- [ ] Finance calls the matched service endpoint and uses response context
- [ ] Enrichment timeout (5s) doesn't block the pipeline; falls back to regex-only
- [ ] Enrichment result is logged on the transaction (new `enrichment_data` JSONB column)

### Scenario 2.2: No Receipt Found

```
GIVEN a raw transaction: {payee: "AMZN MKTP US", amount: -23.47, date: "2025-02-10"}
WHEN finance asks email for a receipt
AND email returns: {found: false}
THEN finance proceeds with regex-only categorization
AND the entry gets confidence 0.7 from ACCOUNT_HINTS pattern match
AND the entry is created as pending (0.7 < 0.8 threshold)
```

**Acceptance criteria:**
- [ ] Missing receipt doesn't raise an error
- [ ] Transaction still gets processed with whatever confidence the rules provide
- [ ] `enrichment_data` records `{attempted: true, found: false}`

### Scenario 2.3: Email Service Down

```
GIVEN the email service is offline or unreachable
WHEN finance tries to enrich a transaction
THEN the enrichment step times out in 5 seconds
AND finance logs a warning but continues processing
AND the transaction is categorized with regex-only (degraded mode)
```

**Acceptance criteria:**
- [ ] Router/email call has a 5-second timeout
- [ ] Timeout is caught, logged, and doesn't propagate
- [ ] Pipeline completion is not blocked by email outage

### Commands (Increment 2)

```bash
# Router query (already built in reeves-router)
curl -X POST localhost:8787/router/api/query -d '{"need": "receipt for $450 Hilton charge on Feb 3"}'

# Email receipt search (to be built in reeves-email)
curl localhost:8787/email/api/emails/receipt?merchant=Hilton&date=2025-02-03&amount=450

# Finance enrichment (new internal function, not a CLI command)
# Called automatically during auto-process when confidence < 0.8 (ENRICH_TRIGGER_THRESHOLD)
```

### Boundaries (Increment 2)
- Finance MUST NOT import email modules directly — all cross-service calls go through HTTP
- Enrichment is opportunistic: failure = fallback to regex, never = pipeline failure
- Receipt data stored as `enrichment_data` JSONB, not parsed into separate columns
- Router calls use the gateway (localhost:8787/router/) not direct service ports

---

## Increment 3: LLM-Powered Categorization

**What:** When regex rules and email receipts still leave confidence below threshold, use `claude -p` to reason about the transaction and suggest an account.

### Scenario 3.1: LLM Categorization for Unknown Transaction

```
GIVEN a raw transaction: {payee: "SQ *BLUE BOTTLE", amount: -6.50, date: "2025-02-12"}
AND no rule matches (confidence 0.0)
AND no receipt found from email
WHEN the pipeline's LLM categorization step runs
THEN finance calls `claude -p` with:
  "Categorize this transaction for a personal chart of accounts.
   Payee: SQ *BLUE BOTTLE
   Amount: -$6.50
   Description: PURCHASE
   Available accounts: [Expenses:Food:Restaurants, Expenses:Food:Coffee, ...]
   Respond with JSON: {account: '...', confidence: 0.0-1.0, reasoning: '...'}"
AND claude responds: {account: "Expenses:Food:Coffee", confidence: 0.9, reasoning: "Blue Bottle is a specialty coffee chain"}
AND the entry is created with account Expenses:Food:Coffee, confidence 0.9, auto-approved
AND a new rule is suggested: pattern="blue\s*bottle", target=Expenses:Food:Coffee
```

**Acceptance criteria:**
- [ ] New `llm_categorize()` function in categorizer.py
- [ ] Uses `claude -p` subprocess (NEVER anthropic SDK) with `--output-format text --max-turns 1`
- [ ] Strips CLAUDECODE and CLAUDE_CODE_ENTRYPOINT env vars before subprocess call
- [ ] Prompt includes the full chart of accounts (or top-level categories)
- [ ] LLM response is parsed as JSON with account + confidence + reasoning
- [ ] LLM categorization is only attempted when rule confidence < 0.5 (don't waste API calls)
- [ ] LLM result stored in enrichment_data alongside any receipt data
- [ ] Successful LLM categorization triggers rule suggestion for future matching

### Scenario 3.2: LLM Fallback on Parse Error

```
GIVEN claude returns non-JSON or an invalid account name
THEN finance logs the raw response
AND falls back to Expenses:Uncategorized with confidence 0.0
AND the entry is created as pending
```

**Acceptance criteria:**
- [ ] JSON parse errors are caught and logged, not raised
- [ ] Invalid account names are caught (not in chart of accounts) and logged
- [ ] Transaction still gets an entry (never dropped)

### Scenario 3.3: LLM Rate Limiting

```
GIVEN the pipeline has 50 transactions needing LLM categorization
THEN finance processes them sequentially with a 1-second delay between calls
AND caps LLM calls at 20 per pipeline run
AND remaining transactions are left for next run
```

**Acceptance criteria:**
- [ ] LLM calls are rate-limited (1/sec) and capped per run (default 20)
- [ ] Cap is configurable via CLI flag `--llm-limit 20`
- [ ] Transactions beyond the cap are simply skipped (not errored)

### Commands (Increment 3)

```bash
# New
reeves-finance categorize TRANSACTION_ID [--json]  # Manual LLM categorization for one txn

# Modified
reeves-finance auto --llm-limit 20  # Enables LLM for low-confidence items
reeves-finance run-pipeline --llm-limit 20  # Same, in pipeline mode
```

### Boundaries (Increment 3)
- MUST use `claude -p` subprocess, NEVER `anthropic` SDK
- MUST strip CLAUDECODE env vars before subprocess call
- LLM is LAST RESORT: rules → email enrichment → LLM (in that order)
- LLM calls are rate-limited and capped per pipeline run
- LLM suggestions auto-create rules so the same pattern doesn't need LLM twice
- LLM errors NEVER cause transaction drops — always fall back to Uncategorized/pending

---

## Increment 4: Daily Summary + Review Queue

**What:** After each pipeline run, generate a human-readable summary. Surface items needing review.

### Scenario 4.1: Daily Pipeline Report

```
GIVEN the nightly pipeline completed
WHEN the summary step runs
THEN it generates a report:
  "Pipeline completed at 2025-02-16 02:03:45
   Synced: 12 new transactions (8 Tiller, 4 Mercury)
   Processed: 12 transactions
   Auto-approved: 9 (confidence >= 0.8)
   Pending review: 3
     - $450.00 HILTON (Expenses:Travel:Hotels, confidence 0.65)
     - $89.99 UNKNOWN VENDOR (Expenses:Uncategorized, confidence 0.0)
     - $1,200.00 WIRE TRANSFER (Transfers:Internal, confidence 0.55)
   Rules created: 2 new rules from patterns
   Balances: Assets $12,450 | Liabilities -$3,200 | Expenses $8,900 (MTD)"
```

**Acceptance criteria:**
- [ ] `reeves-finance report --json` generates the daily summary
- [ ] Report includes pending items with enough context for quick approval/rejection
- [ ] Report includes MTD balance summary
- [ ] Report is written to a file (e.g., `~/.local/share/reeves/reports/finance-YYYY-MM-DD.json`)
- [ ] Web dashboard shows the latest report on the autopilot page

### Scenario 4.2: Review Queue on Dashboard

```
GIVEN there are 3 pending entries
WHEN Dallas opens the finance dashboard
THEN the autopilot page shows:
  - Last pipeline run time and status (green/yellow/red)
  - Pending review items with one-click approve/reject
  - Each pending item shows: date, payee, amount, suggested account, confidence, enrichment context
  - Approve sends HTMX request, removes item from queue
  - Reject prompts for reason, sends HTMX request
```

**Acceptance criteria:**
- [ ] Autopilot page loads pending entries with enrichment context
- [ ] One-click approve/reject via HTMX (no page reload)
- [ ] Pipeline status indicator (last run time, health)
- [ ] Sorted by amount descending (review big items first)

---

## Implementation Sequence

```
Increment 1 (Scheduled Pipeline)
├── 1a. Add `run-pipeline` CLI command (chains sync → auto)
├── 1b. Add pipeline result tracking (write last result to DB or file)
├── 1c. Update `status` to include pipeline stats
├── 1d. Add daemon job scheduler (cron-style, new module in reeves-daemon)
└── 1e. Register finance pipeline as a scheduled job

Increment 2 (Email Enrichment)
├── 2a. Add `enrichment_data` JSONB column to raw_transactions
├── 2b. Build `enrich_transaction()` in entry_builder.py
├── 2c. Wire enrichment into auto-process pipeline
├── 2d. Build receipt search endpoint in reeves-email (prerequisite)
└── 2e. Integration test: transaction → router → email → enriched entry

Increment 3 (LLM Categorization)
├── 3a. Build `llm_categorize()` in categorizer.py
├── 3b. Wire into auto-process as fallback after rules + enrichment
├── 3c. Auto-suggest rules from successful LLM categorizations
└── 3d. Rate limiting and cap configuration

Increment 4 (Daily Summary)
├── 4a. Build `report` CLI command
├── 4b. Wire report generation into pipeline tail
├── 4c. Update autopilot dashboard page
└── 4d. Add one-click approve/reject for pending items
```

**Start with Increment 1.** It's entirely within reeves-finance + reeves-daemon, uses only existing code, and immediately reduces manual work. The 3,003 unprocessed transactions are the test case.

---

## Testing Strategy

Each increment gets its own test file: `tests/spec_increment_N.py`

```bash
# Run all spec tests
pytest tests/spec_*.py -v

# Run one increment
pytest tests/spec_increment_1.py -v
```

Tests use the existing DB fixtures (session injection) and mock external services (email, router) for increments 2+. Increment 1 tests can run against a test database with seed data.

---

## Guardrails

- **No money moves without Ledger validation.** Every entry passes `ledger balance` before storage.
- **No silent failures.** Every transaction gets an entry — worst case is Uncategorized/pending.
- **No concurrent pipelines.** File lock prevents double-runs.
- **Confidence threshold is sacred.** Below 0.8 = pending review. Always.
- **Soft deletes only.** Never DELETE FROM.
- **Session injection.** CLI is composition root. No global DB state.
- **`claude -p` only.** Never import anthropic SDK.
