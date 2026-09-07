# Kimss control plane — what decision makers get

**Audience:** CISO, CTO, security architecture, compliance, procurement, and business owners evaluating Kimss.  
**Purpose:** A single brief of the **control-plane capabilities** customers receive when agent and model traffic runs through Kimss — not an engineering runbook.  
**Product stance:** Kimss is the **enterprise agent control plane**. Your models and tools stay the **data plane**. Kimss governs identity, tools, spend, safety, and audit on the routed hop.

| Surface | Where |
|---------|--------|
| Guardrails | [App → Guardrails](https://kimss.ai/app/guardrails) · [docs](https://kimss.ai/docs/trust_safety) |
| Threat Intercepts | [App → Intercepts](https://kimss.ai/app/intercepts) |
| Agents + kill switch | [App → Agents](https://kimss.ai/app/agents) |
| Provider Vault (BYO) | [App → Vault](https://kimss.ai/app/vault) |
| Team & Access | [App → Workspace](https://kimss.ai/app/workspace) |
| Trust Center | [kimss.ai/trust](https://kimss.ai/trust) |
| Security & compliance docs | [kimss.ai/docs/security_compliance](https://kimss.ai/docs/security_compliance) |
| Integrator OpenAPI | [`openapi/control-plane.yaml`](../openapi/control-plane.yaml) |

---

## 1. One-sentence value

When an agent or SDK call goes through Kimss, you get a **governed gateway**: you decide which agents may run, which tools may fire, what argument values are allowed, what content is safe, what may leave the boundary (web / MCP), how secrets are held, and how blocked or accepted traffic is audited — **without** Kimss hosting your inference or training on your prompts.

---

## 2. Mental model: control plane vs data plane

```text
  Apps / agents / SDKs
           │
           ▼
  ┌─────────────────────────────┐
  │  Kimss control plane        │
  │  Identity · Guardrails      │
  │  Tool policy · Vault        │
  │  Metering · Audit           │
  └─────────────┬───────────────┘
                │ vaulted route
                ▼
  Your providers / MCP / search   ← data plane (you own)
```

| You keep | Kimss provides |
|----------|----------------|
| Model choice and provider bills | Routing with write-only vaulted keys |
| Customer-hosted MCP servers | Opt-in proxy + RBAC + audit |
| Retrieval / RAG on your side | Tool and argument policy at execution time |
| SIEM / NOC destinations | Threat Intercepts + optional export / webhooks |

**Hermis** is the Kimss execution loop that enforces these controls mid-run (kill switch, tool allowlist, argument policies, Authority Boundary, content safety / PII, web/MCP gates) before tools or completions proceed.

---

## 3. Guardrails

Configure under **Governance → Guardrails**. Blocked or flagged calls appear under **Threat Intercepts** (Production+).

### 3.1 Workspace baseline

| Control | What you get | Tier |
|---------|--------------|------|
| **Content safety & prompt attacks** | Classifiers for Hate, Violence, Sexual, Self-harm, and user prompt attacks. Per-category Low / Medium / High / Off. Blocks return a clear explanation (API clients may see **HTTP 451**). Fail-closed if the safety upstream fails. | All plans when enabled |
| **PII & secret scrubbing** | In-process scan for emails, phones, cards, SSNs, IPs, and common API keys / JWTs — **no extra model cost**. Alert = log and continue; Block = deny. Findings go to Threat Intercepts. | **Production+** to enable |
| **Authority Boundary** | Sensitive tool args marked for provenance may only use values a **trusted human** named — not values that only appeared in tool output or retrieved data. Soft denial; intercept as argument taint. | **Production+** to enable |

### 3.2 Agent tools (workspace opt-in)

Egress starts **off** and requires explicit admin consent.

| Control | What you get |
|---------|--------------|
| **Web Search** | Agents cannot attach web search until the workspace opts in. Search runs on **provider** infrastructure — treat as egress in your risk review. |
| **Internal MCP servers** | Agents cannot call customer-hosted MCP until opt-in. Kimss proxies HTTPS MCP with vaulted headers, kill-switch checks, and RBAC. |

### 3.3 Tool argument policies

Guardrails decide **which** tools can run. Argument policies decide **what values** may be passed (for example email must match `@yourco.com`).

- Constraint types: exact match, allowlist, regex, deny pattern, max length, require provenance, optional default-deny for unmapped names.
- Soft-fail: structured tool result so the agent can self-correct.
- Rejected calls → Threat Intercepts.
- **Configure / edit: Production+.** Existing policies still enforce on Developer if already present.

---

## 4. Threat Intercepts & monitoring

| Capability | Tier |
|------------|------|
| Threat Intercepts dashboard (sanitized excerpts, not a full prompt archive) | **Production+** |
| SIEM export (JSON/CSV) | **Scale / Enterprise** |
| Live intercept webhooks (`threat_intercept.recorded`) | **Enterprise** |
| Activity (governed-request log) | All plans |
| Identity / admin audit log | Admin (+ group grant) |
| Agent Tracking (call-site topology) | Shipped |

**Violation types include:** content safety, prompt injection, safety check failed, web search block, MCP block, PII scrub, argument policy, Authority Boundary / argument taint.

---

## 5. Execution control

| Capability | What you get |
|------------|--------------|
| **Agent kill switch** | Disable → subsequent routed calls refuse (**403**). Re-checked mid-hop. |
| **Tool allowlist** | Only tools declared on the agent may execute. |
| **Code interpreter** | AST-gated Python sandbox (no network/files; short timeout). |
| **HTTP webhook tools** | Fixed URL + vaulted auth; model supplies JSON body only. |
| **Article 12–oriented tool audit** | Always on for routed Hermis tool use. |
| **Shadow / JIT discovery** | Agents from gateway traffic can be inventoried; Enterprise can alert. |
| **External agents** | Inventory / report supported; kill switch is authoritative **only when traffic routes through Kimss**. |

---

## 6. Secrets, spend, and FinOps

| Capability | What you get |
|------------|--------------|
| **Provider Vault (BYO)** | Keys envelope-encrypted; **client write-only**; decrypt in memory for routing only. |
| **Per-endpoint token caps** | Monthly cap with alert or block (**429**). |
| **Governed-request metering** | Control-plane allowance; hard cap or overage by plan. Inference stays on your provider. |
| **Retention purge** | Automatic purge of telemetry / intercepts by plan window. |
| **Enterprise alerts** | Webhooks for usage thresholds, threat intercepts, shadow agents. |

**Default telemetry posture:** metadata and token counts — not a full prompt archive.

---

## 7. Identity, access, and tenancy

| Capability | Tier notes |
|------------|------------|
| SSO (Entra / workforce) | Emphasized from Scale; Enterprise multi-tenant onboarding |
| Workspace RBAC | Unlimited Team & Access seats: **Production+** |
| IAM groups & entitlements | Enterprise-oriented |
| SCIM 2.0 Users + Groups | **Enterprise** (Scale+ called out for SCIM token) |
| API keys (`kimss_...`) | All plans |
| Workspace isolation (`X-Workspace-ID`) | All plans |
| Dedicated schema isolation | **Enterprise** |

---

## 8. Plan matrix

Indicative packaging — confirm live terms on [Pricing](https://kimss.ai/pricing) or with sales.

| Capability | Developer (free) | Production | Scale | Enterprise |
|------------|------------------|------------|-------|------------|
| Guardrails soft gate, web/MCP opt-in, kill switch, vault, Agents | Yes | Yes | Yes | Yes |
| Governed requests (included) | 25k hard | 100k + overage | 1M + overage | Custom |
| Telemetry / intercept retention | 14 days | 30 days | 90 days | Custom / unlimited |
| Threat Intercepts UI | No | Yes | Yes + export | Custom retention + SIEM pipelines |
| PII scrub / Authority Boundary / edit tool policies | No* | Yes | Yes | Yes |
| Team & Access (unlimited members) | Limited seats | Yes | Yes | Yes |
| Entra SSO / domain routing | — | — | Yes | Yes |
| SCIM, schema isolation, live intercept webhooks, Enterprise Alerts | — | — | Partial / marketed | Yes |

\*Developer still **enforces** argument policies if they already exist; configuration UI and paid safety toggles require Production+.

---

## 9. Compliance & evidence (buyer language)

Not a substitute for your DPA or attestations.

1. **Gateway-verified audit** — Article 12–oriented immutable gateway logs, distinct from storing full prompts.
2. **Always-on tool audit** — who ran what tool when on the routed hop.
3. **Payload transience** — write-only keys; sanitized intercept excerpts; plan-bound retention.
4. **Explicit egress consent** — web search and internal MCP require workspace opt-in.
5. **Fail-closed safety** — safety upstream failure does not silently allow harmful traffic.
6. **Customer data plane** — you choose models and MCP hosts.

Trust Center: [https://kimss.ai/trust](https://kimss.ai/trust). Attestations under NDA via Enterprise sales where applicable.

---

## 10. What is intentionally *not* included

| Topic | Reality |
|-------|---------|
| Hosted File Search / RAG in Kimss | **Retired.** Retrieve on your side; govern tools/MCP at the gateway. |
| Kimss as the model host | Not the product — BYO / vaulted endpoints. |
| Full prompt archive as SIEM | Not the default. |
| Kill switch for off-gateway agents | Authoritative only when traffic routes **through** Kimss. |
| Web search content inspection | Kimss gates attach-time access; query/result processing is on the provider. |

---

## 11. Talking points by role

**CISO / Security** — Content safety, prompt-attack shielding, optional PII/secret scrub, Authority Boundary, default-deny web/MCP egress, Threat Intercepts, SIEM-ready export on higher tiers — without cleartext model keys in Kimss.

**CTO / Platform** — One control plane in front of vaulted OpenAI-compatible or Anthropic endpoints. Tool allowlists and argument policies mid-loop. Kill switch and governed-request metering first-class. OpenAPI + SDKs in this hub.

**Compliance / Privacy** — Opt-in scrubbing, retention windows, Article 12–oriented tool and gateway audit. DPA and agreement remain authoritative.

**Procurement / Finance** — Free Developer proves the gateway. Production unlocks Threat Intercepts, Team & Access, and argument-policy management. Scale/Enterprise add retention, SSO/SCIM, and SIEM pipelines. You pay Kimss for **governed requests**, not provider tokens.

**Business owner** — Agents stay useful; unsafe content, secret leakage, and unauthorized tool values get blocked or logged before they become an incident.

---

## 12. Suggested 30-minute demo

1. **Guardrails** — safety thresholds, PII, Authority Boundary (Production+).  
2. **Web Search / MCP** — off by default + consent.  
3. Sample **tool argument policy** (e.g. email domain regex).  
4. Trigger a block → **Threat Intercepts**.  
5. **Kill switch** — disable agent, show refused call.  
6. **Vault** — write-only keys narrative.  
7. Close with [Trust Center](https://kimss.ai/trust) + plan matrix.

---

## Related links in this hub

| Doc | Use |
|-----|-----|
| [README](../README.md) | Integrator quickstart + surface summary |
| [`openapi/control-plane.yaml`](../openapi/control-plane.yaml) | Registry, MCP grants, audit, metering, kill switch |
| [AI_INTEGRATION.md](../AI_INTEGRATION.md) | Agent-to-agent / coding-agent contract |
| [examples/](../examples/) | MCP RBAC and governance policy examples |

*Last updated: 2026-09-07. Align commercial claims with [pricing](https://kimss.ai/pricing) and Enterprise proposals before external distribution.*
