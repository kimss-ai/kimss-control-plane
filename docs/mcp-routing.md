# Internal MCP routing — secure proxy to your MCP servers

Kimss is the **secure proxy** between your vaulted model and **customer-hosted** Model Context Protocol (MCP) servers. Tool calls are attributed to workspace identity, checked against kill switch + Guardrails + RBAC (+ optional argument policies), then executed with vaulted headers that never appear in list/GET responses.

**This is not** the optional [Kimss Python SDK MCP server](https://kimss.ai/docs/python_sdk_mcp) (`kimss-mcp-server` for Cursor/IDE → Kimss API). That path exposes Kimss *as* tools. This page is **your MCP servers behind Kimss** (Hermis → your HTTPS MCP).

**Product doc:** https://kimss.ai/docs/routing_internal_mcp_servers  
**OpenAPI:** [`openapi/control-plane.yaml`](../openapi/control-plane.yaml) (tag `mcp`)  
**A2A / Cascade contract:** [`AI_INTEGRATION.md`](../AI_INTEGRATION.md)

---

## Cascade / coding-agent procedure (MCP)

When the user asks to register or govern an internal MCP server through Kimss:

1. **Confirm Guardrails opt-in** — Internal MCP must be **On** under `/app/guardrails` (workspace consent). Off → register/attach/execute blocked.
2. **Register the server** (admin) — `POST /api/v1/mcp-servers` with public **HTTPS** `base_url` (no localhost / RFC1918). Auth headers are **write-only**. Example: [`examples/mcp-server-register.json`](../examples/mcp-server-register.json).
3. **Discover tools** — `POST /api/v1/mcp-servers/{server_name}/discover`.
4. **Attach / allowlist on the agent** — in the Kimss UI (Agents → Tools → Attach MCP) enable only the tools that agent may call. Hermis injects them as `mcp__{server}__{tool}`.
5. **Optional RBAC grants** — if any `iam.mcp_tool_grants` rows exist for the server, they become authoritative. Upsert via `PUT .../grants` with bodies like [`examples/mcp-tool-grant-admin-only.json`](../examples/mcp-tool-grant-admin-only.json).
6. **Optional argument policies** — constrain tool argument *values* under Guardrails (Production+ to edit). Violations soft-fail; MCP `tools/call` does not run.
7. **Do not** put MCP bearer tokens in the customer app repo — vault them on register/rotate only.
8. **Verify** — run a Hermis/agent turn that should call an allowed tool; confirm audit / Agents activity. Kill-switch the agent → further MCP hops must fail with `agent_disabled`.

Inference traffic still follows [`AI_INTEGRATION.md`](../AI_INTEGRATION.md) (native SDK → `api.kimss.ai` + Gateway key + `X-Kimss-Agent-Id`).

---

## Register (curl)

```bash
curl -sS -X POST "https://api.kimss.ai/api/v1/mcp-servers" \
  -H "Authorization: Bearer kimss_..." \
  -H "Content-Type: application/json" \
  -d @examples/mcp-server-register.json
```

Rotate credentials later:

```bash
curl -sS -X POST "https://api.kimss.ai/api/v1/mcp-servers/crm-tools/rotate" \
  -H "Authorization: Bearer kimss_..." \
  -H "Content-Type: application/json" \
  -d '{"auth_headers":{"Authorization":"Bearer NEW_TOKEN"}}'
```

List responses never include decrypted headers (`has_credential` / `credential_write_only` only).

---

## Enforcement stack (order)

1. **Kill switch** — disabled agent stops Hermis mid-hop (including MCP).
2. **Guardrails** — `mcp_enabled` workspace opt-in required.
3. **Agent tool allowlist** — only toggled `mcp__{server}__{tool}` names reach the model.
4. **MCP tool grants** — if any grants exist for the server, they gate who may call which tool (`role` | `oid` | `group`).
5. **Tool argument policies** — value constraints before `tools/call`.
6. **Audit** — `tool_execute` + zero-token telemetry (`mcp-tool-*`); parent model turn counts as one **governed request**.

---

## Troubleshooting

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Cannot register / 403 MCP | Guardrails Internal MCP off or non-admin | Enable under `/app/guardrails`; use admin key/session |
| SSRF / invalid URL | localhost or private IP | Expose public HTTPS (e.g. tunnel); Kimss rejects RFC1918 |
| Tool never appears to the model | Not discovered or not attached on agent | Discover + attach/allowlist in Agents UI |
| Tool call denied | Grant missing / wrong principal | Upsert grant or remove grants to fall back to role defaults |
| `argument_policy_violation` | Argument policy blocked values | Adjust policy or payload; see Threat Intercepts (Production+) |
| `agent_disabled` | Kill switch | Re-enable agent under `/app/agents` |

---

## Related

- [`examples/mcp-server-register.json`](../examples/mcp-server-register.json)
- [`examples/mcp-tool-grant-*.json`](../examples/)
- Decision-maker overview: [decision-maker-brief.md](decision-maker-brief.md) (Internal MCP opt-in)
- IDE → Kimss tools (different path): https://kimss.ai/docs/python_sdk_mcp
