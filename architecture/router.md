# The Router: Capability Discovery

The router knows what every service can do. Agents ask it questions instead of knowing the topology themselves. New services register capabilities and the whole system gets smarter without changing existing code.

## The Problem

Without a router, cross-service communication is N-squared. Finance needs to know email's API to get receipts. Tax needs to know finance's API to get transactions. Health needs to know relationships' API to identify doctors. Every new service has to learn about every existing one.

## The Solution

One service that maintains a capability map. Any agent or service that needs data describes what it needs. The router responds with where to get it and how.

```
Finance job: "I have a $450 Hilton charge from Feb 3. I need context."

Router: [
  { service: "email", action: "search", params: { merchant: "Hilton", date_range: "Feb 1-5" } },
  { service: "relationships", action: "lookup", params: { name: "Hilton", type: "organization" } }
]
```

Finance doesn't know email or relationships exist. It asked a question and got a plan.

## Starting Point: Finance ↔ Email Enrichment

The router's first test case is concrete: when finance pulls transactions, it needs to find matching emails (receipts, confirmations, invoices) to link to each transaction. This is the simplest cross-service data flow in the system:

1. Finance job pulls new transactions on schedule
2. For each transaction, finance asks the router: "I need context for a $450 Hilton charge on Feb 3"
3. Router returns: email can search by merchant + date, relationships can identify the merchant
4. Finance calls email's search endpoint, gets back matching receipt
5. Receipt is linked to the transaction

This is the test case. If the router can make this flow clean, it'll work for everything else. Don't over-scope — prove it here first.

## Why Build Early

The router isn't about replacing existing MCP tools or slimming the kernel. It's about making cross-service data flows possible without hardcoding service-to-service knowledge.

1. **Clean cross-service flows from day one.** Finance doesn't need to import email's client library or know its API shape.
2. **Every new service is immediately available.** Register capabilities, done. No changes to existing services.
3. **Token-friendly.** When an agent needs cross-service data, one `ask_router` call returns only relevant capabilities instead of scanning the full tool registry. The router narrows the field.

This pattern didn't exist pre-AI. Old routers (Consul, Istio, API gateways) route by address — you have to know what you want and where it lives. This routes by need. That requires understanding the question and matching it to capabilities. Pre-AI, that would have been a massive hand-coded rules engine. Now it's a capability registry plus one inference call.

## Dynamic Capability Lookup

The router is like a dynamic tool. Static tools say "here are 100 things I can do, pick one." The router says "what do you need?" and returns only the relevant capabilities.

```
# What an agent or job sees
ask_router:
  description: "Describe what you need. Returns which services can help and how to call them."
  params:
    need: string   # Natural language description of what you're trying to accomplish
```

The caller doesn't need to know the topology. It describes the need, gets back a focused set of actions, and executes them.

## Capability Registration

Each service declares what it can answer:

```yaml
# In capabilities.yaml
services:
  email:
    description: "Gmail sync, AI triage, search, draft responses"
    capabilities:
      - name: get_receipt
        description: "Find receipt or confirmation email matching a transaction"
        endpoint: "/api/emails/receipt"
        params: [merchant, date, amount]
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

## Relationship to MCP

Claude already has MCP tool dispatch — register tools, model picks which to call. The router doesn't replace MCP tools. It's for service-to-service and job-to-service communication where the caller doesn't know the topology. The tools still exist. The router is an additional path, not a replacement.
