# The Router: Capability Discovery

The router knows what every service can do. Agents ask it questions instead of knowing the topology themselves. New services register capabilities and the whole system gets smarter without changing existing code.

## The Problem

Without a router, cross-service communication is N-squared. Finance needs to know email's API to get receipts. Tax needs to know finance's API to get transactions. Health needs to know relationships' API to identify doctors. Every new service has to learn about every existing one.

## The Solution

One service that maintains a capability map. Any agent or service that needs data describes what it needs. The router responds with where to get it and how.

```
Agent: "I have a $450 Hilton charge from Feb 3. I need context."

Router: [
  { service: "email", action: "search", params: { merchant: "Hilton", date_range: "Feb 1-5" } },
  { service: "assets", action: "store", params: { type: "receipt", source: "email" } },
  { service: "relationships", action: "lookup", params: { name: "Hilton", type: "organization" } }
]
```

Finance doesn't know email, assets, or relationships exist. It asked a question and got a plan.

## Capability Registration

Each service declares what it can answer:

```yaml
# In service's reeves.yaml or equivalent
capabilities:
  - name: search_transactions
    description: "Find financial transactions by merchant, date, amount, category"
    params: [merchant, date_range, amount_range, category]

  - name: get_balance
    description: "Current balance for an account"
    params: [account_name]
```

The router aggregates these into a system-wide capability map. When a query comes in, it matches the need to available capabilities.

## Where It Lives in the Loop

```
INFER decides WHAT to know
  → Router decides HOW to know it
    → Daemon ensures the services are RUNNING to answer
      → Mechanic verifies the answers ARRIVED correctly
```

The router is INFER's execution layer. It doesn't decide what matters — that's the brain. It decides where to go to find out.

## Implementation Timing

Build after the first real cross-service flow (finance ↔ email enrichment) ships. That integration will reveal the actual interface patterns. Router-first would be speculative. Router-after-one-real-integration is informed.

## Relationship to MCP

Claude already has MCP tool dispatch — register tools, model picks which to call. The router is the same pattern for service-to-service communication, running locally without an LLM in the loop for every routing decision. The capability map is static knowledge, not inference.
