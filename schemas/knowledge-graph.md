# Knowledge Graph — RDF Triples with Confidence

## What It Is

Reeves' knowledge graph stores relationships between any entities as subject-predicate-object triples, inspired by RDF. It enables traversal, context gathering, and relationship discovery across all domains.

The graph is polymorphic — subjects and objects can be any entity type (person, task, project, fact, intent, organization, place). Predicates are standardized strings. Confidence scores indicate how certain we are about each relationship.

## Schema

```sql
CREATE TABLE reeves.relationship_edges (
    id              SERIAL PRIMARY KEY,

    -- Subject (from entity)
    subject_type    VARCHAR NOT NULL,   -- 'person', 'task', 'project', 'fact', 'intent'
    subject_id      VARCHAR NOT NULL,   -- ID as string for polymorphism

    -- Predicate (relationship type)
    predicate       VARCHAR NOT NULL,   -- 'IS_CHILD_OF', 'BLOCKED_BY', 'WORKS_AT'

    -- Object (to entity)
    object_type     VARCHAR NOT NULL,
    object_id       VARCHAR NOT NULL,

    -- Metadata
    confidence      FLOAT DEFAULT 0.8,  -- 0.0-1.0 certainty
    source          TEXT,               -- evidence or provenance
    valid_from      TIMESTAMPTZ,        -- temporal validity start
    valid_to        TIMESTAMPTZ,        -- temporal validity end (NULL = current)

    -- Standard columns
    created_at      TIMESTAMPTZ DEFAULT NOW(),
    updated_at      TIMESTAMPTZ DEFAULT NOW(),
    deleted_at      TIMESTAMPTZ,        -- soft delete

    -- Constraints
    UNIQUE (subject_type, subject_id, predicate, object_type, object_id)
);
```

## Standard Predicates

### Family Relationships
- `IS_CHILD_OF` — Parent-child relationship
- `IS_PARENT_OF` — Reverse of IS_CHILD_OF
- `IS_SIBLING_OF` — Sibling relationship
- `IS_SPOUSE_OF` — Spouse/partner relationship

### Work Relationships
- `WORKS_AT` — Employment
- `MANAGES` — Management
- `REPORTS_TO` — Reporting chain
- `COLLABORATES_WITH` — Collaboration

### Task/Project Relationships
- `BLOCKED_BY` — Dependency/blocker
- `BLOCKS` — Reverse dependency
- `PART_OF` — Containment (task part of project)
- `DEPENDS_ON` — General dependency
- `RELATED_TO` — General relationship

### Location Relationships
- `LOCATED_IN` — Physical location
- `LIVES_IN` — Residence
- `WORKS_IN` — Work location

### Ownership/Association
- `OWNS` — Ownership
- `CREATED_BY` — Creator
- `ASSOCIATED_WITH` — General association
- `MEMBER_OF` — Group membership
- `INTERESTED_IN` — Interest/topic connection

## Confidence Rules

**Before ANY insert/update/delete:**

1. Devlog the evidence first
2. Clues are not bells — shared last name, same city are clues, not proof
3. Multi-source preferred — 2+ sources get `confidence >= 0.9`

| Confidence | Meaning | Example |
|-----------|---------|---------|
| `< 0.7` | Hypothesis (needs verification) | "Might work at X based on email domain" |
| `0.7-0.9` | Probable (single strong source) | "User said they work at X in voice session" |
| `> 0.9` | Confirmed (multiple sources or explicit) | "LinkedIn + user confirmation" |

**Source field is mandatory.** Use `evidence:<description>` format, never just `inference`.

## Traversal Operations

### `graph_traverse(start_type, start_id, predicates, depth)`
Walk the graph from a starting node, following specified predicates up to N hops.

Example: "Who does Daniel know through work?"
```
traverse(person, dallas_id, [WORKS_AT, COLLABORATES_WITH], depth=2)
```

### `graph_context(entity_type, entity_id)`
Gather all direct relationships (1-hop) around an entity. Used by mindreader to understand context.

### `graph_find_path(from_type, from_id, to_type, to_id)`
Find shortest path between two entities across any predicates.

### `graph_add_edge(subject, predicate, object, confidence, source)`
Add a new relationship with mandatory evidence.

## How the Graph Grows

### From Observation (Sleep Cycles)
Sleep daemon analyzes patterns and creates edges:
- "User messages Person X every Sunday" → `Person(user) COMMUNICATES_WITH Person(X)` with temporal pattern
- "Task A always follows Task B" → `Task(A) DEPENDS_ON Task(B)`

### From Explicit Input
User tells Reeves or creates Webb relationships:
- "Colleen is my daughter" → `Person(Colleen) IS_CHILD_OF Person(Daniel)`
- Manual person/org/place creation in Webb

### From AI Inference
Brain module infers relationships from data:
- Cross-referencing message contacts with calendar attendees
- Linking finance transactions to known organizations

## Temporal Validity

Relationships have time bounds:
- `valid_from` — When the relationship started
- `valid_to` — When it ended (NULL = still current)

This enables historical queries: "Who did you work with in 2024?" vs "Who do you work with now?"

## Implementation

Source: `reeves/graph.py` (673 lines)

Key functions:
- `add_edge()` — Create a relationship triple
- `traverse()` — Walk the graph from a starting node
- `gather_context()` — All 1-hop relationships for an entity
- `find_path()` — Shortest path between two entities
- `delete_edge()` — Soft delete a relationship
