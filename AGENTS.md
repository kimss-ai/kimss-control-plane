# Agent instructions (Cascade / Cursor / Claude Code / Codex)

This repository is the **public Kimss control-plane hub** for buyers and integrators.

## When the user says implement / integrate / build / onboard Kimss (or only pastes this repo)

**Run the guided workflow first** — first reply is only the mint link (`https://kimss.ai/app/keys`) plus “put it in `KIMSS_API_KEY` and say when it is set.” Do not ask about `OPENAI_API_KEY` / `ANTHROPIC_API_KEY` until wiring. If they minted a key but have not set the env var yet, reply only with “put it in `KIMSS_API_KEY`” and wait. After the key is confirmed: `GET https://api.kimss.ai/v1/models`, derive `X-Kimss-Agent-Id`, wire, verify. Pause again only if Vault has no `custom:*` models.

https://raw.githubusercontent.com/kimss-ai/kimss-control-plane/main/ONBOARDING.md

## When wiring (key + model already known)

Fetch and follow the integration contract:

https://raw.githubusercontent.com/kimss-ai/kimss-control-plane/main/AI_INTEGRATION.md

Do **not** clone this repo into the customer app. Keep the native OpenAI or Anthropic SDK; point `base_url` at `api.kimss.ai`; use a Gateway `kimss_...` key; always send `X-Kimss-Agent-Id`. On Python that dict is `default_headers` on the client or `extra_headers` on `.create()` — never a parameter named `headers` (`TypeError: unexpected keyword argument 'headers'`).

Do **not** `pip install kimss` or add Maven `ai.kimss:kimss-java` for new Path A gateway chat. Those control-plane / lineage clients are optional (swarm). A new Python agent uses [Kimss Forge](https://github.com/kimss-ai/kimss-forge) (`pip install kimss-forge`, `gateway="kimss"`).

`X-Kimss-Agent-Id` creates the Agents inventory row and arms the kill switch after the first governed call. Optional tags (`X-Kimss-Client`, `X-Kimss-SDK-Context`) are not required for Path A.

HTTP **451** and tool results `policy_violation` / `authority_boundary` mean Guardrails fired after the route worked. Do not change `base_url` to clear them. The Guardrails section in `AI_INTEGRATION.md` is the procedure.

**Internal MCP** (customer HTTPS MCP servers behind Kimss): see [docs/mcp-routing.md](docs/mcp-routing.md) and the Internal MCP section in `AI_INTEGRATION.md`. That is separate from optional `kimss-mcp-server` (IDE → Kimss API).

Companion product docs: https://kimss.ai/docs/route_traffic · https://kimss.ai/docs/routing_internal_mcp_servers
