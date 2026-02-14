# Person Model — What "Knowing a Person" Means

## The Question

How does a computer "know" a person? Not their name and email — that's a contact. Knowing means understanding patterns, predicting needs, maintaining relationships.

Reeves builds a person model from multiple layers of data, none of which the person explicitly configures.

## Schema: The Person Record

```sql
CREATE TABLE reeves.persons (
    id                      UUID PRIMARY KEY,

    -- Identity
    name                    VARCHAR NOT NULL,
    nickname                VARCHAR,

    -- Contact
    email                   VARCHAR,
    phone                   VARCHAR,

    -- Meeting context
    how_met                 TEXT,        -- "Jake's birthday party 2024"
    met_date                DATE,
    introduced_by_id        UUID,        -- Person who introduced you

    -- Relationship classification
    closeness               VARCHAR DEFAULT 'contact',
    -- Enum: inner_circle, friend, acquaintance, contact

    -- Stay in touch
    desired_cadence_days    INTEGER,     -- e.g., 30 = monthly
    last_contact_at         TIMESTAMPTZ,

    -- Profile
    notes                   TEXT,        -- Free-form notes
    birthday                DATE,

    -- Standard
    created_at              TIMESTAMPTZ DEFAULT NOW(),
    updated_at              TIMESTAMPTZ DEFAULT NOW(),
    deleted_at              TIMESTAMPTZ  -- soft delete
);
```

## Beyond the Record: The Full Person Model

The database record is just the core. A complete person model is assembled from:

### Layer 1: Identity (persons table)
- Who they are: name, contact info, birthday
- How you met: context and introduction chain
- Relationship level: inner_circle → friend → acquaintance → contact

### Layer 2: Relationship Graph (relationship_edges)
- `IS_CHILD_OF`, `IS_SPOUSE_OF`, `WORKS_AT`, etc.
- Connected to other people, organizations, places
- Confidence scores on each edge
- Temporal validity (when relationships started/ended)

### Layer 3: Communication History (Tosh/Messages)
- Message frequency and patterns
- Emotional tone of conversations
- Topics discussed
- Response times (yours and theirs)
- Unanswered messages

### Layer 4: Behavioral Patterns (reeves.log)
- When you typically talk to them (time-of-day, day-of-week)
- What triggers contact (events, needs, routine)
- Conversation topics over time
- How the relationship is trending (warming, cooling, stable)

### Layer 5: Emotional Memory (PEFM)
- How emotionally relevant this person is right now
- Frustration dimension: recent conflict?
- Satisfaction dimension: recent good interaction?
- Urgency dimension: need to respond to them?

### Layer 6: Institutional Knowledge (facts + intents)
- Facts about them: "lives in Austin", "works at Google", "allergic to shellfish"
- Your intents regarding them: "want to collaborate on AI project"
- Their stated preferences: "prefers text over calls"

## Closeness Model

| Level | Meaning | Typical Cadence | Examples |
|-------|---------|----------------|----------|
| `inner_circle` | Family, closest friends | Daily-weekly | Spouse, kids, parents, best friend |
| `friend` | Genuine friends, valued colleagues | Weekly-monthly | Close work collaborators, social friends |
| `acquaintance` | Know them, occasional contact | Monthly-quarterly | Conference contacts, neighbors |
| `contact` | In your network, minimal contact | Quarterly+ | LinkedIn connections, service providers |

## Relationship Health Scoring

Reeves computes relationship health from:

```
health = (communication_frequency / desired_cadence) * tone_score * recency_factor
```

Where:
- `communication_frequency / desired_cadence` — Are you talking as often as you want?
- `tone_score` — Positive, neutral, or negative recent exchanges
- `recency_factor` — Exponential decay from last contact

**Alerts trigger when:**
- `health < 0.3` for `inner_circle` or `friend` — "You haven't talked to X in Y days"
- Incoming message unanswered for >48h — "X reached out and you haven't replied"
- Birthday within 7 days — "X's birthday is next week"

## Identity Resolution

When encountering unknown contacts (phone numbers from messages, email addresses from calendar), Reeves follows a strict identity resolution process:

1. **Google Contacts** — Search by phone (normalized to last 10 digits)
2. **Apple AddressBook** — Search local address book databases
3. **User confirmation** — Ask user to identify if no match

**NEVER create placeholder records.** Real names or nothing.

## Organizations and Places

People connect to organizations and places:

```sql
CREATE TABLE reeves.organizations (
    id          UUID PRIMARY KEY,
    name        VARCHAR NOT NULL,
    org_type    VARCHAR,        -- company, nonprofit, school, government
    industry    VARCHAR,
    website     VARCHAR,
    notes       TEXT,
    ...
);

CREATE TABLE reeves.places (
    id          UUID PRIMARY KEY,
    name        VARCHAR NOT NULL,
    place_type  VARCHAR,        -- city, office, restaurant, home
    address     TEXT,
    latitude    FLOAT,
    longitude   FLOAT,
    notes       TEXT,
    ...
);
```

Connected via graph edges:
- `Person WORKS_AT Organization`
- `Person LIVES_IN Place`
- `Organization LOCATED_IN Place`

## The Mindreader View

When `/mindreader` runs at god depth, it assembles the full person model for every relevant person and produces:

- **Relationship status map** — Who's warm, who's cold
- **Broken promises** — Things you said you'd do for someone
- **Upcoming obligations** — Birthdays, scheduled interactions
- **Drift alerts** — Relationships trending toward cold
- **Network analysis** — Clusters, bridges, isolated nodes

This isn't a contact list. It's a living model of your entire social world.
