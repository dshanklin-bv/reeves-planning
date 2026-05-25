# The Nervous System: Why Reeves Owns Its Data

*Why an autonomous system must internalize its world, not query it on every thought.*

---

## The Design Choice

When building Reeves, every data source presented the same fork in the road:

**Option A:** Jobs query the source directly. Need to read a text thread? Hit the iMessage database. Need to check email? Call the Gmail API. The system reaches into the environment each time it needs information.

**Option B:** A daemon continuously syncs source data into a Reeves-owned database. Jobs read from the internal store. The system builds and maintains its own representation of the world.

Option A is simpler to start. Option B is the right architecture. And the reason why has nothing to do with engineering tradeoffs. It has to do with what kind of system you're building.

## Organisms Don't Query the Environment

Your brain doesn't send a signal to the outside world every time it needs to know the temperature. It has thermoreceptors in your skin that continuously feed data into your nervous system. Your brain maintains an internal model of the thermal environment. When you reach for a cup, you already know whether it's hot — not because you queried reality in that moment, but because your nervous system has been passively maintaining that model since the cup was placed in front of you.

This isn't a performance optimization. It's a *requirement* for autonomous behavior.

An organism that has to query the environment before every action is reactive. It can only respond to what it asks about. It has no peripheral awareness, no passive pattern recognition, no ability to notice things it wasn't looking for.

An organism with its own nervous system — continuously sensing, normalizing, maintaining an internal model — can do something qualitatively different: it can *think*. It can cross-reference signals from different senses. It can detect patterns over time. It can notice anomalies. It can anticipate. It can act on its own model of the world without waiting for a prompt.

Reeves is the second kind of system.

## The Sync Daemon Is the Nervous System

The messages sync daemon reads from iMessage's ChatDB every few minutes. It normalizes the data — handles, threads, attachments, timestamps — and writes it into Reeves's own PostgreSQL schema. The email sync does the same for Gmail. Future adapters will do the same for WhatsApp, Telegram, Calendar, Photos.

Each sync daemon is a sensory nerve. It translates raw environmental signal (the gnarly format of ChatDB, the paginated Gmail API, the binary blob of a photo attachment) into normalized internal signal that the rest of the system can process.

The database is the brain's representation. Not the raw world — a *model* of the world, maintained continuously, queryable instantly, cross-referenceable across domains.

When a job runs — "triage the last hour of messages" — it doesn't parse ChatDB. It queries a clean, indexed, normalized table. The job operates on the *model*, not on reality. Just like your brain operates on its neural representation of the world, not on the photons hitting your retina.

## Why This Matters for Autonomy

Five consequences that only exist because Reeves owns its data:

### 1. Cross-Domain Awareness

"Show me all messages from people I have overdue tasks about."

This query joins the messages schema with the planner schema. It's trivial SQL over two normalized tables. If messages lived in ChatDB and tasks lived in a separate app, this query would require two different data access patterns, two different schemas, and a reconciliation step. With an internalized model, it's one query.

This is analogous to cross-modal integration in the brain. You can recognize a friend's voice on the phone because your auditory cortex and your social memory share a unified representation. The raw signals (sound waves, visual memories) are different formats from different senses. The brain's internal model normalizes them into something that can be cross-referenced.

### 2. Temporal Awareness

"What's new since the last triage?"

With an internal model: `WHERE synced_at > :last_run`. One indexed query.

Without an internal model: the job would have to remember what it saw last time, query the source again, diff the results. Every job becomes a stateful integration problem. The state management that the sync daemon handles once, elegantly, would have to be reimagined for every job type.

### 3. Source Independence

iMessage stores data in SQLite with a schema Apple changes without notice. Gmail's API has rate limits, pagination, and authentication token management. Each source is a snowflake of complexity.

The sync daemon absorbs that complexity. It's the only code that knows ChatDB has a `handle_id` that maps to a `chat_handle_join` that maps to a `chat`. Jobs never see this. They see `conversations` and `messages` in a clean schema.

When Apple changes ChatDB (and they do, regularly), one daemon gets updated. Every job, every routine, every piece of intelligence built on top — unchanged. The nervous system adapted. The brain didn't notice.

### 4. Availability

iMessage data is local-only. Gmail requires network. Both can be unavailable — disk unmounted, network down, API rate limited.

Reeves's database is always available. Jobs don't stall because a source is temporarily unreachable. The model may be slightly stale (last sync was 5 minutes ago), but it's always there. An organism whose brain goes offline every time one sense is blocked wouldn't survive long.

### 5. The Foundation for Intelligence

Pattern recognition, anomaly detection, relationship tracking, commitment extraction — every piece of intelligence in Reeves operates on the internal model. None of it queries raw sources.

This isn't an implementation detail. It's a prerequisite. You cannot build intelligence on raw, heterogeneous, intermittently-available data sources. You need a unified, normalized, always-available representation first. The sync layer makes intelligence *possible*. Without it, every intelligent feature would have to solve the data access problem from scratch, and most wouldn't be built at all.

## The Cockpit Reads Gauges, Not Sensors

This connects directly to the cockpit model.

A cockpit has gauges. Altitude, airspeed, heading, engine temperature. The pilot reads gauges. The pilot does not read raw sensor output — the pitot tube pressure differential, the gyroscope precession rate, the thermocouple voltage from the exhaust gas.

The sync daemon is the sensor-to-gauge translation layer. Raw ChatDB is the pitot tube. The normalized `messages.conversations` table is the airspeed indicator. The pilot (or in Reeves's case, the jobs and the intelligence layer) reads the gauge.

This separation is what makes the cockpit usable. A cockpit that showed raw sensor data would be incomprehensible. A system where jobs parsed raw source data would be fragile, slow, and impossible to build intelligence on top of.

## ETL Is the Nervous System Pattern

In data engineering, this pattern has a name: ETL (Extract, Transform, Load). Extract from sources, transform into a useful shape, load into a store. It's the oldest pattern in data architecture.

What's novel here isn't the pattern. It's recognizing that ETL is a *biological necessity* for autonomous systems, not just an engineering convenience.

Every autonomous system — biological or artificial — that operates on information from its environment has a nervous system. Something that continuously senses, normalizes, and maintains an internal representation. The specific technology doesn't matter (PostgreSQL, neurons, whatever). The pattern is invariant: **internalize the world, then think about the internalized model.**

Systems that skip this step — that query the environment on every thought — are not autonomous. They're reactive. They can only respond to what they're asked about. They cannot notice, anticipate, cross-reference, or learn. They are tools waiting for instructions, not organisms living in a world.

## The Cost

There is a cost. The sync daemon is real infrastructure. It needs to run continuously. It needs error handling for source failures. It needs schema management as sources change. It needs monitoring to ensure freshness.

In biology, the nervous system is 2% of body mass but consumes 20% of the organism's energy. It's expensive. But organisms without nervous systems — sponges, plants — don't move, don't hunt, don't plan, and don't think. They survive by being rooted and reactive.

The cost of a nervous system is the price of autonomy. Reeves pays it.

## The Rule

**Reeves never operates on raw external data.** Every data source flows through a sync daemon into a Reeves-owned schema. Jobs, routines, intelligence, and the cockpit all read from the internal model. The sync layer is the only code that touches external sources.

This is not a technical preference. It is a biological requirement for the kind of system Reeves is.
