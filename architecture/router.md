# The Router: Capability Discovery

The router knows what every service can do. Agents ask it questions instead of knowing the topology themselves. New services register capabilities and the whole system gets smarter without changing existing code.

## The Problem

Without a router, cross-service communication is N-squared. Finance needs to know email's API to get receipts. Tax needs to know finance's API to get transactions. Health needs to know relationships' API to identify doctors. Every new service has to learn about every existing one.

There's also a cost problem. reeves-web (the kernel) currently has 100+ MCP tools loaded. Every agent call forces the model to scan all of them to pick the right one. More tools in context = more tokens per call = slower routing = more expensive. The kernel is the de facto router today — it just does it expensively, by putting everything in context and hoping the model picks correctly.

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

## Why Build Early

The router pays for itself immediately:

1. **Token reduction.** Instead of 100 tool definitions in every agent context, one `ask_router` tool. The model sends a need, gets back only the relevant capabilities. Smaller context = faster inference = cheaper.
2. **Kernel slimming.** reeves-web currently acts as the router by knowing about every module. Extract the routing concern and the kernel gets simpler.
3. **Clean cross-service flows from day one.** The finance ↔ email enrichment pipeline doesn't need to be built direct and then refactored. It goes through the router from the start.
4. **Every new service is immediately available.** Register capabilities, done. No changes to existing services or the kernel.

This pattern didn't exist pre-AI. Old routers (Consul, Istio, API gateways) route by address — you have to know what you want and where it lives. This routes by need. That requires understanding the question and matching it to capabilities. Pre-AI, that would have been a massive hand-coded rules engine. Now it's a capability registry plus one inference call.

## Dynamic Capability Lookup

The router is like a dynamic tool. Static tools say "here are 100 things I can do, pick one." The router says "what do you need?" and returns only the relevant capabilities.

One tool in the MCP registry. Every service behind it.

```
# What an agent sees in its tool list
ask_router:
  description: "Describe what you need. Returns which services can help and how to call them."
  params:
    need: string   # Natural language description of what you're trying to accomplish
```

The agent doesn't need to know the topology. It describes the need, gets back a focused set of actions, and executes them. The 100-tool context window becomes a 1-tool context window with targeted follow-up.

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

## Relationship to MCP

Claude already has MCP tool dispatch — register tools, model picks which to call. The router doesn't replace MCP tools. It means an agent's first call is "what can help me with X" instead of scanning 100 tool definitions. The tools still exist. The router narrows the field.
