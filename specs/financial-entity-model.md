# Spec: Financial Entity Model

**Goal:** Every transaction maps to a real source account and a known merchant. Dirty bank payee strings resolve to clean entities. The pipeline uses this to categorize accurately instead of dumping everything into Uncategorized.

**Status:** Draft
**Service:** reeves-finance

---

## What Exists

### Tables (schema built, some empty)

| Table | Rows | Status |
|-------|------|--------|
| `finance.accounts` | 65 | Populated — full chart of accounts with hierarchy (parent) and types |
| `finance.merchants` | 0 | Schema exists: id, name, display_name, merchant_type, default_account, entity_id |
| `finance.merchant_aliases` | 0 | Schema exists: merchant_id, pattern, match_field, priority |
| `finance.categorization_rules` | 25 | Populated — regex rules on payee/description → target account |
| `reeves.institutions` | 0 | Schema exists: name, institution_type, website, routing_number |
| `reeves.entities` | ? | Legal entities (LLCs, etc.) — kernel owns CRUD |

### Web UI (built)

| Page | Status | Notes |
|------|--------|-------|
| Institutions CRUD | Built | `web/routers/institutions.py` — full create/update/delete |
| Merchants CRUD + aliases | Built | `web/routers/merchants.py` — create/update/delete + alias patterns |
| Entities read-only | Built | `web/routers/entities.py` — reads from kernel, links to detail |

### Source Account Mapping (broken)

`entry_builder.py:_get_source_account()` hardcodes three sources:
```python
if source == 'mercury': return 'Assets:Bank:Mercury'
elif source == 'chase': return 'Liabilities:CreditCard:Chase'
elif source == 'amex': return 'Liabilities:CreditCard:Amex'
else: return DEFAULT_ACCOUNTS['checking']  # Mercury fallback
```

**Problem:** ALL 3,113 transactions come from Tiller (source='tiller'), not 'mercury'/'chase'/'amex'. The `account_id` field has the real info:

```
tiller / platinum_card_(-1002)           → 1,011 txns  (Amex Platinum)
tiller / d._shanklin_(-8165)             →   743 txns  (Chase checking or credit)
tiller / everyday_checking_...4604       →   411 txns  (Wells Fargo checking)
tiller / credit_card_(-1261)             →   156 txns  (Chase credit card)
tiller / platinum_card                   →   155 txns  (Amex Platinum)
tiller / business_checking_...3897       →   136 txns  (Mercury business)
tiller / blue_cash_preferred             →   117 txns  (Amex Blue Cash)
tiller / blue_cash_preferred_(-1002)     →    87 txns  (Amex Blue Cash)
tiller / venture                         →    32 txns  (Capital One Venture)
tiller / usaa_djs_checking               →    26 txns  (USAA checking)
tiller / mortgage_...9781                →    17 txns  (Wells Fargo mortgage)
... and 14 more
```

None of these resolve to a chart-of-accounts entry. Every transaction defaults to `Assets:Bank:Mercury`.

### Payee Normalization (missing)

Raw payees from Tiller are dirty:
- `"Tst* Palm Wood - Brafort Worth TX"` → should be "Palm Wood" (restaurant)
- `"Purchase, Authorized On, 02/03 Domino's x, x-x-2222, Tx, Sx9364, Card 4026"` → should be "Domino's"
- `"Py *blue Skies Storaboone NC"` → should be "Blue Skies Storage"
- `"Awardwalle* Order 18bethlehem PA"` → should be "AwardWallet"

No normalization runs before categorization. The ACCOUNT_HINTS patterns in `analyzer.py` would match "domino's" and "storage" but categorization only checks the 25 DB rules, not ACCOUNT_HINTS.

---

## The Entity Relationships

```
Institution (bank/card issuer)
  └── Source Account (a specific account at that institution)
        └── maps to → Chart of Accounts entry (Assets:Bank:Chase, Liabilities:CreditCard:Amex, etc.)

Merchant (normalized vendor)
  └── Aliases (dirty payee patterns that resolve to this merchant)
        └── default_account → Chart of Accounts entry (Expenses:Food:Restaurants, etc.)

Raw Transaction
  ├── source + account_id → Source Account → Chart of Accounts (debit/credit side)
  └── payee (dirty) → Merchant Alias → Merchant → default_account (category side)
```

---

## Slices (elephant carpaccio)

### Slice 1: Source Account Map

**What:** Map Tiller `account_id` values to chart-of-accounts entries. Fix `_get_source_account()`.

```
GIVEN a raw transaction with source='tiller', account_id='platinum_card_(-1002)'
WHEN the entry builder runs
THEN the source account is 'Liabilities:CreditCard:Amex:Platinum'
NOT 'Assets:Bank:Mercury'
```

**Implementation:** Add a `finance.source_accounts` table or a simple dict/YAML mapping. The 25 Tiller account_ids map to ~10 chart-of-accounts entries. This is a static lookup, not AI.

**Acceptance:**
- [ ] All 25 Tiller account_id values have mappings
- [ ] `_get_source_account()` uses the mapping
- [ ] Re-run `auto --limit 10` → source accounts are correct
- [ ] Unknown account_ids fall back to Expenses:Uncategorized with a logged warning

**Test:** `reeves-finance auto --limit 10 --json` → check postings have correct source accounts.

### Slice 2: Seed Institutions

**What:** Create institutions for the banks Daniel uses. Link to source accounts.

```
GIVEN the institutions table is empty
WHEN we seed it with known banks
THEN:
  - Chase (bank + credit_card)
  - Amex (credit_card)
  - Wells Fargo (bank + mortgage)
  - Mercury (bank)
  - USAA (bank)
  - Capital One (credit_card)
  - Citi (credit_card)
```

**Implementation:** Migration script or CLI command. 7 institutions, each with type.

**Acceptance:**
- [ ] `reeves.institutions` has 7 rows
- [ ] Dashboard institutions page shows them
- [ ] Each institution links to its source accounts

**Test:** Dashboard shows institutions. `reeves-finance status --json` includes institution count.

### Slice 3: Payee Normalizer

**What:** Clean dirty payee strings before categorization. Extract the merchant name from bank noise.

```
GIVEN payee = "Purchase, Authorized On, 02/03 Domino's x, x-x-2222, Tx, Sx9364, Card 4026"
WHEN the normalizer runs
THEN normalized_payee = "Domino's"

GIVEN payee = "Tst* Palm Wood - Brafort Worth TX"
WHEN the normalizer runs
THEN normalized_payee = "Palm Wood"

GIVEN payee = "Py *blue Skies Storaboone NC"
WHEN the normalizer runs
THEN normalized_payee = "Blue Skies Storage"
```

**Implementation:** `normalize_payee()` function in a new `reeves_finance/payee.py` module. Rules:
1. Strip known prefixes: `TST*`, `SQ *`, `PY *`, `Purchase, Authorized On, ...`
2. Strip location suffixes: city + state at end
3. Strip card/transaction IDs: `x-x-NNNN`, `Card NNNN`, `SNNNN`
4. Title case the result

**Acceptance:**
- [ ] Handles the 10 real payees from our test run
- [ ] Runs before categorization in `build_entry_from_transaction()`
- [ ] Existing rules that matched on raw payee still work (normalizer doesn't break them)

**Test:** Unit tests with the 10 real payees. Then `reeves-finance auto --limit 10 --json` → payees are clean.

### Slice 4: Merchant Auto-Discovery

**What:** When the pipeline processes a transaction and the normalized payee doesn't match any merchant, auto-create a merchant + alias.

```
GIVEN normalized_payee = "Domino's"
AND no merchant has an alias matching "Domino's"
WHEN the transaction is processed
THEN a new merchant is created: {name: "Domino's", merchant_type: "vendor"}
AND a merchant_alias is created: {pattern: "domino", match_field: "payee"}
AND next time a Domino's transaction appears, it resolves to this merchant
```

**Implementation:** After normalization, check `merchant_aliases` for a match. If no match, create merchant + alias. If merchant has `default_account`, use it for categorization.

**Acceptance:**
- [ ] Processing 50 transactions auto-creates merchants for each unique normalized payee
- [ ] Dashboard merchants page shows discovered merchants
- [ ] Merchant aliases match future transactions
- [ ] No duplicate merchants for the same vendor

**Test:** `reeves-finance auto --limit 50 --json` → merchants table has N rows. Run again → 0 new merchants created.

### Slice 5: Merchant → Account Mapping

**What:** When a merchant has a `default_account`, use it for categorization instead of regex rules.

```
GIVEN merchant "Domino's" has default_account = "Expenses:Food:Delivery"
AND a new Domino's transaction arrives
WHEN the categorizer runs
THEN the entry is categorized as Expenses:Food:Delivery with confidence 0.95
AND the entry is auto-approved (0.95 >= 0.8)
```

**Implementation:** In `categorize_transaction()`, check merchant_aliases → merchant → default_account BEFORE checking categorization_rules. Merchant match confidence = 0.95 (higher than regex rules at 0.7).

**Acceptance:**
- [ ] Merchant-based categorization takes priority over regex rules
- [ ] Merchant match confidence = 0.95 (auto-approve eligible)
- [ ] Merchants without default_account fall through to regex rules
- [ ] Dashboard lets you set default_account on discovered merchants

**Test:** Set default_account on one merchant via dashboard. Run pipeline. That merchant's transactions get the right account.

### Slice 6: Wire ACCOUNT_HINTS into Categorizer

**What:** The 30+ ACCOUNT_HINTS patterns in `analyzer.py` are only used during `analyze`, not during `categorize_transaction()`. Wire them in as a fallback.

```
GIVEN payee contains "domino's"
AND no merchant alias matches
AND no categorization rule matches
WHEN ACCOUNT_HINTS check runs
THEN "restaurant" pattern matches → Expenses:Food:Restaurants with confidence 0.7
```

**Implementation:** In `categorize_transaction()`, after checking rules and before returning None, check ACCOUNT_HINTS. This is the existing logic from `analyzer._suggest_account()` — just needs to be called in the categorization path too.

**Acceptance:**
- [ ] ACCOUNT_HINTS patterns used as fallback in categorize_transaction()
- [ ] Confidence from ACCOUNT_HINTS = 0.7 (same as current)
- [ ] Rules still take priority (checked first)

**Test:** `reeves-finance auto --limit 50 --json` → fewer Uncategorized entries.

---

## Slice Sequence

```
Slice 1: Source Account Map        → fixes every transaction's debit/credit side
Slice 2: Seed Institutions         → populates the entity model
Slice 3: Payee Normalizer          → clean data for matching
Slice 4: Merchant Auto-Discovery   → builds the merchant registry from real data
Slice 5: Merchant → Account Map    → merchants drive categorization
Slice 6: Wire ACCOUNT_HINTS        → immediate coverage boost from existing patterns
```

Each slice has one test: run `auto --limit N` and check the output. Run it, see what changed, decide what's next.

---

## Guardrails

- **Merchant discovery is additive.** Auto-create only. Never auto-delete or merge merchants.
- **Source account mapping is static.** Known account_ids → known accounts. Unknown → fallback + warning.
- **Normalizer must not break existing rules.** Rules that matched on raw payee must still work. Run normalizer in addition to raw matching, not instead of.
- **Soft deletes only.** All entities use `deleted_at`.
- **Finance owns finance.* tables.** Institutions are in `reeves.*` (kernel-owned). Finance reads them, doesn't write.
