# Source: https://support.originhq.com/docs/origin-mcp-server

Origin's Model Context Protocol (MCP) server lets you query your endpoint and agent observability data from any MCP-compatible client — Claude Desktop, Claude Code, Cursor, Cline, and others. Ask natural-language questions about your AI agent activity, prompt usage, model spend, and endpoint behavior, and the LLM you're using will translate them into structured queries against your Origin tenant.

This guide covers what the server does, how to connect, sample prompts, common errors, and how to get support.

---

## 

What the MCP server gives you

Origin's MCP exposes read-only access to your analytics graph — the same data that powers the Origin dashboard. From an MCP client you can answer questions like:

- Which agents are running on which endpoints, and what models are they calling?
- How much are we spending on each model, broken down by department or team?
- What prompts fail most often, and where?
- Which sessions had the longest tool-call chains yesterday?

The server is read-only. It cannot modify endpoints, agents, alerts, or any other Origin state.

## 

Tools exposed

The server exposes two tools. Most MCP clients will discover them automatically once connected.

### 

`analytics_describe_schema`

Returns the full analytics graph: entities, fields, metrics, join edges, the query DSL reference, and worked examples. Takes no parameters.

Use this when an LLM needs to learn the shape of your data before composing a query. Most clients call it implicitly the first time you ask a data question.

### 

`analytics_query`

Executes a structured query against your tenant's data and returns rows plus query cost.

Parameters (the LLM fills these in for you):

- `entity` — the root entity to query (e.g. `endpoints`, `ai_prompts`, `sessions`)
- `joins` — how to traverse into related entities
- `filters` — predicates applied before aggregation
- `select` — dimension fields to return (optionally bucketed by time)
- `metrics` — aggregations to compute (counts, sums, percentiles, computed metrics)
- `order_by`, `limit`, `offset` — paging and sort
- `time_range_days` — defaults to 30
- `sample` — distribution-preserving sampling for expensive queries

The response includes the result rows, a `truncated` flag, and a `cost` object showing how many rows and bytes were scanned, how long the query took, and whether it used a primary-key anchor. Queries that exceed your tenant's cost budget are rejected with a suggestion to narrow the scope.

## 

Connect a client

The server speaks Streamable HTTP and uses OAuth 2.1 with Dynamic Client Registration. Any MCP client that supports remote HTTP MCP servers can connect.

**Endpoint:** `https://mcp.prod.originhq.com/mcp`

**Authentication:** OAuth 2.1 via your Origin login (WorkOS AuthKit). The first time you connect, your client will pop a browser window for sign-in. After that, refresh tokens are handled automatically by your client.

You do not need to provision API keys, copy bearer tokens, or paste anything from the Origin dashboard. The OAuth flow handles everything.

### 

Claude Desktop

Add this to your `claude_desktop_config.json` (Settings → Developer → Edit Config):

JSON

```
{
  "mcpServers": {
    "origin": {
      "url": "https://mcp.prod.originhq.com/mcp"
    }
  }
}
```

Restart Claude Desktop. The first time you reference Origin in a chat, you'll be prompted to sign in.

### 

Claude Code

Bash

```
claude mcp add --transport http origin https://mcp.prod.originhq.com/mcp
```

Then run `claude` and complete the sign-in prompt the first time you use the server.

### 

Cursor

Settings → MCP → Add new MCP server:

JSON

```
{
  "mcpServers": {
    "origin": {
      "url": "https://mcp.prod.originhq.com/mcp"
    }
  }
}
```

### 

Cline / other clients

Any client that supports a remote MCP server over HTTP can connect. Point it at `https://mcp.prod.originhq.com/mcp` and follow your client's instructions for OAuth sign-in.

## 

Sample prompts

Once connected, you can ask questions in plain English. Examples to try:

- "What models are my Claude Code agents calling this week, and on which endpoints?"
- "Show me total token spend by department over the last 30 days, sorted highest first."
- "Which prompts are failing most often? Group by model and tool name."
- "Find the ten longest agent sessions yesterday and show me which tools each one called."
- "Compare prompt latency p50 and p95 across our models for the past two weeks."
- "Which endpoints had AI activity for the first time this week?"
- "How many Claude Code sessions did each engineer run this week?"
- "Show me MCP server connections by endpoint — which agents are connecting to which servers?"

For the best results, give the LLM enough specificity to compose a query: include a time range, a grouping dimension, and what you want to measure. If the LLM asks clarifying questions, answer them — it's usually trying to disambiguate between similar fields.

## 

Troubleshooting

### 

"I added the server but no Origin tools appear in my client"

Most often this means the client hasn't finished discovery. Fully quit and reopen the client. If the tools still don't show up, check the client's MCP logs for connection errors and confirm the endpoint URL is exactly `https://mcp.prod.originhq.com/mcp` with no trailing path.

### 

"Sign-in window doesn't open" or "stuck on authorize"

Your client needs an OAuth-capable browser flow. Make sure you're on a recent version — older Claude Desktop, Cursor, and Cline builds do not support OAuth 2.1 Dynamic Client Registration. Update and retry.

### 

"Authorization failed" / 401 errors after sign-in

Your token may have expired and your client may not be refreshing it. Disconnect and reconnect the server in your client's settings. If the problem persists, your Origin account may not have access to the tenant you're trying to query — confirm in the Origin dashboard that you can see the data there first.

### 

"Validation error" / "DSL → query" errors in responses

The LLM composed a query the server couldn't validate — usually a field referenced on the wrong entity, or a filter value of the wrong type. These come back as visible errors so the LLM can self-correct on the next turn. If it loops, try restating your question with the field names you saw in the schema, or ask the LLM to "call `analytics_describe_schema` again before retrying."

### 

"Cost exceeds budget" errors

You asked for something that would scan more rows than your tenant's per-query budget allows. The error message includes a suggestion (usually: narrow the time range, add a filter, or sample the result). Time-bound your question more tightly and try again.

### 

Empty results when you expected data

The LLM may have applied a filter that doesn't match anything. Ask it to "show the query it ran" — most clients will print the DSL — and check the filters. Common culprits: a hostname filter with the wrong case, or a time range that excludes the data you care about.

### 

Slow responses

Analytics queries scale with the data they scan. If a question consistently takes more than 10–20 seconds, try narrowing the time range or asking for an aggregate (counts, top-N) instead of raw rows.

For anything not covered here, see [support.originhq.com](https://support.originhq.com) or contact us (below).

## 

Versioning and protocol

- **MCP protocol version:** `2025-06-18` (with fallback to `2025-03-26` and `2024-11-05`)
- **Transport:** Streamable HTTP (POST `/mcp`)
- **Auth:** OAuth 2.1 with Dynamic Client Registration (RFC 7591) and Protected Resource Metadata (RFC 9728)
- **Request body limit:** 256 KiB

Discovery metadata is published at `https://mcp.prod.originhq.com/.well-known/oauth-protected-resource` for clients that need it. You should not need to configure this manually.

## 

Data, privacy, and audit

Every tool call is recorded to Origin's audit log with the calling user, the tool name, a hash and snapshot of the arguments, the row count, bytes scanned, elapsed time, and outcome. Audit records are scoped per tenant and retained for 30 days. Queries run with your user identity — you can only see what your Origin role allows you to see in the dashboard.

## 

Get support

- **In-app:** Open the support panel in the Origin dashboard.
- **Email:** [support@originhq.com](mailto:support@originhq.com)
- **Docs:** [support.originhq.com](https://support.originhq.com)

When reporting an issue, include your MCP client and version, the prompt you ran, and any error message the client returned. If the server returned a structured error, the message is usually enough for us to pinpoint the problem.

Copy Page