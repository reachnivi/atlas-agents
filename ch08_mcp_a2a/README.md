# Chapter 8 — Open Protocols: MCP and A2A

> **Atlas v0.8** — expose a knowledge base as an **MCP server** so any MCP client can use it, and publish an **A2A Agent Card** so other agents can discover and delegate to Atlas.

## TL;DR

Two open protocols replace custom glue code:

- **MCP (Model Context Protocol)** connects an agent to **tools and data**. You write a server once and every MCP client (Claude Desktop, Claude Code, IDEs, your own agent) can use it.
- **A2A (Agent-to-Agent)** connects an agent to **other agents**. An agent publishes an *Agent Card* at `/.well-known/agent.json` that lists its skills, and other agents send it *tasks*.

```
 Agent ──MCP──▶ Tools/Data (your server)          Agent A ──A2A task──▶ Agent B
                                                      │  GET /.well-known/agent.json
                                                      └─ discover skills first
```

## Key concepts

| Concept | What it means | Where to see it |
|---|---|---|
| **MCP server** | Declares tools with `@server.list_tools()` and handles calls with `@server.call_tool()`. | `mcp_knowledge_server.py` |
| **Transport** | `stdio_server()`: the client starts the server as a subprocess and talks over stdin/stdout. | `main()` |
| **Tool guardrails in the server** | `query_knowledge` rejects anything that isn't `SELECT`, and `search_knowledge` uses parameterized SQL. | `call_tool()` |
| **FastMCP** | A decorator-style shortcut for small MCP servers. | `online/mcp_github_server.py` |
| **Agent Card** | JSON describing name, URL, version, capabilities, and skills (with input/output modes). | `AGENT_CARD`, written to `.well-known/agent.json` |
| **Discovery → delegation** | Fetch cards, match a skill by keyword, send a task, read the text artifact. | `online/a2a_client_discovery.py` |
| **A2UI** | Agents emit structured JSON widgets and the UI maps them to trusted components. Never render raw LLM HTML. | `online/a2ui_react_renderer.tsx` |

## What you build

`mcp_knowledge_server.py`, a SQLite-backed knowledge base (seeded with MCP, A2A, LangGraph, CrewAI, and E2B entries) exposing three MCP tools:

| Tool | Purpose |
|---|---|
| `query_knowledge` | Read-only SQL (`SELECT` only) |
| `search_knowledge` | Keyword search over title/content/tags |
| `add_knowledge` | Insert a new entry |

At startup it also writes the A2A Agent Card to `.well-known/agent.json`.

## Files

| File | What it shows |
|---|---|
| `mcp_knowledge_server.py` | The chapter project: MCP server + A2A Agent Card. |
| `online/mcp_github_server.py` | Minimal FastMCP server with a GitHub repo search tool. |
| `online/a2a_client_discovery.py` | A2A client: discover agents, pick the best skill match, delegate a task. |
| `online/a2ui_react_renderer.tsx` | React sketch of safe agent-to-UI rendering. |

## What needs to be done

- [ ] `pip install mcp`.
- [ ] Register the server with an MCP client, for example in Claude Desktop / Claude Code config:
      `{"command": "python", "args": ["/abs/path/ch08_mcp_a2a/mcp_knowledge_server.py"]}`.
- [ ] Ask the client "search the knowledge base for LangGraph", then add an entry and query it back.
- [ ] Try a `DELETE` through `query_knowledge` and confirm it is rejected.
- [ ] Open the generated `.well-known/agent.json` and map each field to an A2A concept.
- [ ] (Stretch) Serve the card over HTTP and point `a2a_client_discovery.py` at it.

## Run it

```bash
cd ch08_mcp_a2a
python mcp_knowledge_server.py        # speaks MCP over stdio; normally launched by an MCP client
python online/a2a_client_discovery.py # pip install httpx
```

## Production notes

- An MCP server runs with **your** credentials. Validate every argument server-side.
- "Only SELECT" string checks are weak; use a read-only database role.
- Agent Cards are public. Don't list internal-only skills in them.
- Chapter 22 uses A2A to plug an external security auditor into the capstone.

## Takeaways

1. MCP standardizes agent-to-tool connections, and A2A standardizes agent-to-agent connections.
2. Write a capability once as an MCP server and every compatible client can use it.
3. Discovery (Agent Cards) lets agents choose collaborators at runtime.

**Prev:** [Ch. 7](../ch07_model_portability/README.md) · **Next:** [Chapter 9 — Agent Skills](../ch09_agent_skills/README.md)
