# Agent instructions (Cascade / Cursor / Claude Code)

This repository is the **public Kimss control-plane hub** for buyers and integrators.

**To wire a customer application to the Kimss gateway, fetch and follow:**

https://raw.githubusercontent.com/kimss-ai/kimss-control-plane/main/AI_INTEGRATION.md

Do **not** clone this repo into the customer app. Keep the native OpenAI or Anthropic SDK; point `base_url` at `api.kimss.ai`; use a Gateway `kimss_...` key; always send `X-Kimss-Agent-Id`.

Do **not** `pip install kimss` or add Maven `com.kimss:kimss-java` for new gateway work. Those control-plane clients are deprecated. A new Python agent uses [Kimss Forge](https://github.com/kimss-ai/kimss-forge) (`pip install kimss-forge`, `gateway="kimss"`).

HTTP **451** and tool results `policy_violation` / `authority_boundary` mean Guardrails fired after the route worked. Do not change `base_url` to clear them. The Guardrails section in `AI_INTEGRATION.md` is the procedure.

**Internal MCP** (customer HTTPS MCP servers behind Kimss): see [docs/mcp-routing.md](docs/mcp-routing.md) and the Internal MCP section in `AI_INTEGRATION.md`. That is separate from optional `kimss-mcp-server` (IDE → Kimss API).

Companion product docs: https://kimss.ai/docs/route_traffic · https://kimss.ai/docs/routing_internal_mcp_servers
