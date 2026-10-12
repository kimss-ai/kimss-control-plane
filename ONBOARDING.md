# Guided onboarding — Kimss gateway

**Agent-agnostic workflow** for Cascade, Cursor, Claude Code, Codex, Windsurf, Devin, and any other coding assistant.

**Fetch URL (prefer this over cloning):**

```text
https://raw.githubusercontent.com/kimss-ai/kimss-control-plane/main/ONBOARDING.md
```

When a user says **implement**, **integrate**, **build**, or **onboard** (Kimss / gateway / control plane) and points at this repo — or only pastes the repo link — **run this workflow in order**. Do **not** invent a Gateway key or vaulted model id. The only default pause is the Gateway key (Step 1–2). Do **not** invent a `custom:` id by prefixing a model name found in the repo.

Do **not** clone this repo into the customer app. Rewire the **customer** codebase only. Wiring contract: [`AI_INTEGRATION.md`](https://raw.githubusercontent.com/kimss-ai/kimss-control-plane/main/AI_INTEGRATION.md).

**Done** means one live HTTP **200** with `X-Kimss-Agent-Id`, then the agent row appears on https://kimss.ai/app/agents and disabling that row is the kill switch. Do **not** claim discovery or kill switch from the key alone.

---

## Step 1 — Ask for the Gateway API key (say this first)

Keep every reply in this step to **a few short sentences**. Do **not** explain Kimss architecture, base URL, vault aliases, headers, SDKs, registry, RBAC, metering, or what will change yet — that comes after the key is in `KIMSS_API_KEY`. Do **not** paraphrase the README.

Do **not** mention `OPENAI_API_KEY` or `ANTHROPIC_API_KEY` here. Those SDK drop-in aliases are only for Step 5 wiring.

### First reply (user just said implement / integrate / build / onboard Kimss / pasted this repo)

They have not confirmed a key yet. Reply with **only** this, then wait:

> Mint a Gateway key at https://kimss.ai/app/keys, put it in `KIMSS_API_KEY`, and say when it is set. Do not paste the full key here.

### If they minted a key but have not set an env var yet

Reply with **only** this, then wait:

> Put that `kimss_…` key in `KIMSS_API_KEY`. Confirm when it is set — do not paste the full key.

**Stop and wait** for confirmation that the key is in `KIMSS_API_KEY` before any other onboarding talk or file edits. Then continue to Step 2.

---

## Step 2 — Confirm the Gateway API key

Once they reply:

1. Expect `KIMSS_API_KEY` (if they already use `KIMSS_WORKSPACE_KEY` or another name for the same `kimss_…` value, accept it and continue — do not quiz them about aliases). Prefer documenting `KIMSS_API_KEY` as the canonical name.
2. Treat “I minted it and put it in `KIMSS_API_KEY`” (or an equivalent prefix confirmation) as enough — do **not** re-ask for the full secret.
3. **Never** paste the full key into chat. **Never** commit it.

If they already confirmed a key (or prefix) in `KIMSS_API_KEY`, move on to Step 3. If they only minted and still need to place it, use the short “put it in env” reply from Step 1 and wait again.

---

## Step 3 — Resolve the vaulted model (prefer API, pause only if empty)

Do **not** invent a `custom:` id. Prefer discovering what the key can already call:

1. Call `GET https://api.kimss.ai/v1/models` with `Authorization: Bearer` equal to the Gateway `kimss_…` key (from `KIMSS_API_KEY` / accepted alias). Do **not** put the full key in chat.
2. Collect `custom:<model_id>` ids from the response (OpenAI-style `data[].id` list).
3. **If exactly one** `custom:*` id — use it and continue. Tell the user which alias you will use.
4. **If several** — ask which exact `custom:…` string to use; wait for their answer. Do not guess.
5. **If none** — this is the only extra pause after the key. Reply briefly and wait:

> Add a model at https://kimss.ai/app/vault, then say when it is saved (or paste the exact `custom:…` alias). Do not invent an id.

**Rules for the agent:**

- Use **only** an id returned by `GET /v1/models` or named by the user from Vault.
- Do **not** invent an id by prefixing `custom:` onto a model name found in the repo.
- A **400** “model not registered” means that id is missing from Vault — stop and use the id on `/app/vault` or re-list with `GET /v1/models`. Do not change `base_url` to clear it.

---

## Step 4 — Agent identity (derive and continue)

Derive a stable slug for `X-Kimss-Agent-Id` from the service or app name (e.g. `acme-shop-assistant`, `billing-bot`). Prefer also `X-Kimss-Agent-Name` (human-readable).

**Do not wait** for confirmation unless the user objects or the slug is ambiguous. Tell them the id you will send, then continue to wiring.

Use one stable id per service so audit, spend, and kill-switch stay attributable. `X-Kimss-Agent-Id` is the header that creates the Agents inventory row and arms the kill switch. Optional client tags (`X-Kimss-Client`, `X-Kimss-SDK-Context`) are **not** required for Path A.

---

## Step 5 — Wire the customer app

Fetch and follow the integration contract (raw preferred):

```text
https://raw.githubusercontent.com/kimss-ai/kimss-control-plane/main/AI_INTEGRATION.md
```

GitHub page: https://github.com/kimss-ai/kimss-control-plane/blob/main/AI_INTEGRATION.md

Summary (details and SDK keyword names are in that file):

1. **Detect** OpenAI-compatible vs Anthropic (or both) in the customer codebase.
2. **Keep** the native client for chat. Do **not** add `KimssClient` / `AgentsApi.run` for inference. New Python agents with no client yet → [Kimss Forge](https://github.com/kimss-ai/kimss-forge) (`gateway="kimss"`).
3. **Base URL** — OpenAI-compatible: `https://api.kimss.ai/v1`. Anthropic: `https://api.kimss.ai` only (no `/v1/messages` suffix on `base_url`).
4. **Key** — Gateway `kimss_…` from `KIMSS_API_KEY` (map into `OPENAI_API_KEY` / `ANTHROPIC_API_KEY` only if the native SDK reads those names). Accept `KIMSS_WORKSPACE_KEY` as the same value when already set.
5. **Model** — exact vaulted alias from Step 3.
6. **Headers** — always send `X-Kimss-Agent-Id` (and preferably `X-Kimss-Agent-Name`). On Python: `default_headers` on the client or `extra_headers` on `.create()`. **Never** pass a parameter named `headers` to `.create()` (`TypeError`).
7. **Forward** Kimss agent headers through any HTTP interceptor / proxy middleware.

---

## Step 6 — Verify (this is when discovery and kill switch become ready)

1. Make one non-stream chat/completions or messages call with the vaulted model and `X-Kimss-Agent-Id`.
2. Expect **200** and a normal assistant payload (not an HTML login page).
3. Tell the user: open **[Agents](https://kimss.ai/app/agents)** (`https://kimss.ai/app/agents`) — the `X-Kimss-Agent-Id` should appear after this first call. **Disabling that row is the kill switch** (next routed call with that id → **403** `agent_disabled`).
4. Optional meter: `GET https://api.kimss.ai/api/v1/governed-requests/meter` with `Authorization: Bearer kimss_…`.

**Guardrails:** HTTP **451** or tool JSON `policy_violation` / `authority_boundary` means the route worked and policy fired. Do **not** change `base_url` or install a Kimss SDK to clear them. See the Guardrails section in `AI_INTEGRATION.md` and https://kimss.ai/docs/trust_safety.

Do not claim success until a live call returns **200** (or the user confirms Vault + key + model alias and a successful call). Do not claim kill switch or agent discovery from the key alone.

---

## Step 7 — Optional: Swarm / multi-agent lineage (only if the user asks)

Path A (Steps 1–6) does **not** require a Kimss package. Swarm governance (Observe/Enforce on [Swarm Runs](https://kimss.ai/app/runs)) needs a **lineage client** that sends `X-Kimss-Run-Id` + child headers. Without that, leave governance **Off** — the gateway still works.

Ask once: “Do you need multi-agent / swarm trees under Guardrails?” If no, stop after Step 6.

If yes, pick **one** (details in `AI_INTEGRATION.md` → Swarm):

| Stack | Setup-once |
|-------|------------|
| New Python agent | [Kimss Forge](https://github.com/kimss-ai/kimss-forge) — `gateway="kimss"` + `Agent.delegate()` |
| Existing Python OpenAI/Anthropic app | `pip install "kimss>=2.2.0"` → `KimssProxy` (auto lineage; not `KimssClient` chat) |
| Java | Maven `ai.kimss:kimss-java` ≥0.3.0 → `KimssProxy` (or forward lineage headers manually) |
| Other / BYO orchestrator | Forward response lineage headers on each hop — Advanced |

Then tell them to set **Observe** or **Enforce** under Guardrails → Agents & Delegation. Demo trees: **Run a demo swarm** on `/app/runs` (no model calls).

---

## Related

| Doc | Role |
|-----|------|
| [AI_INTEGRATION.md](AI_INTEGRATION.md) | Full wiring contract, swarm lineage, troubleshooting, Guardrails |
| [AGENTS.md](AGENTS.md) | Short agent entry |
| [docs/anthropic-onboarding.md](docs/anthropic-onboarding.md) | Anthropic-specific path |
| [docs/mcp-routing.md](docs/mcp-routing.md) | Internal MCP (only if the user asked) |
| https://kimss.ai/docs/route_traffic | Product companion |
| https://kimss.ai/docs/delegation_governance | Swarm / declared-run governance |
