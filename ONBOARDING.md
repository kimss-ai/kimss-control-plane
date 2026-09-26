# Guided onboarding — Kimss gateway

**Agent-agnostic workflow** for Cascade, Cursor, Claude Code, Codex, Windsurf, Devin, and any other coding assistant.

**Fetch URL (prefer this over cloning):**

```text
https://raw.githubusercontent.com/kimss-ai/kimss-control-plane/main/ONBOARDING.md
```

When a user says something like “onboard Kimss” and points at this repo (or only pastes the repo link), **run this workflow in order**. Do **not** invent a Gateway key, vaulted model id, or agent slug. Pause after each step that needs a user answer.

Do **not** clone this repo into the customer app. Rewire the **customer** codebase only. Wiring contract: [`AI_INTEGRATION.md`](https://raw.githubusercontent.com/kimss-ai/kimss-control-plane/main/AI_INTEGRATION.md).

---

## Step 1 — Welcome (say this first)

Relay a short welcome before editing any files. Cover:

1. **What Kimss is** — a model-agnostic enterprise AI gateway and governance control plane. Traffic goes through Kimss; the app keeps its native OpenAI or Anthropic client (or OpenAI-compatible HTTP).
2. **What will change** — base URL → `api.kimss.ai`, API key → a Gateway `kimss_...` workspace key (not the provider key), vaulted model alias `custom:…`, and a stable `X-Kimss-Agent-Id` header. No data-plane refactor; do not add `pip install kimss` / Maven `kimss-java` for chat.
3. **What you need from the user** — a Gateway key and a vaulted model alias (next steps). Agents appear under `/app/agents` after the first governed request; they do **not** need to be created in the UI first.

Then continue to Step 2.

---

## Step 2 — Gateway API key

Ask the user to mint a workspace Gateway key if they do not already have one:

1. Open **[Gateway → Keys](https://kimss.ai/app/keys)** (`https://kimss.ai/app/keys`).
2. Mint a key. It starts with `kimss_…`.
3. Store it as an env var — prefer `KIMSS_API_KEY`. OpenAI-compatible clients often use `OPENAI_API_KEY`; Anthropic clients often use `ANTHROPIC_API_KEY`. Same Gateway key value in either case.
4. **Never** paste the full key into chat. **Never** commit it.

Wait for confirmation: env var name and (optionally) a short prefix such as `kimss_…waYU`. If they already have a key in the environment, accept that and move on.

---

## Step 3 — Vaulted model

Ask which vaulted model alias to use:

1. Open **[Provider Vault](https://kimss.ai/app/vault)** (`https://kimss.ai/app/vault`).
2. Each callable model must exist as `custom:<model_id>` (exact string).
3. If none exists yet: add a vault row for the provider model, note the alias shown in the UI, then tell the agent that exact alias.

**Rules for the agent:**

- Use **only** the alias the user names.
- Do **not** invent an id by prefixing `custom:` onto a model name found in the repo.
- A **400** “model not registered” means that id is missing from Vault — stop and use the id on `/app/vault`. Do not change `base_url` to clear it.

Wait for the user’s exact `custom:…` string before wiring.

---

## Step 4 — Agent identity

Propose a stable slug for `X-Kimss-Agent-Id` from the service name (e.g. `acme-shop-assistant`, `billing-bot`). Prefer also `X-Kimss-Agent-Name` (human-readable).

Ask the user to confirm or override. Use one stable id per service so audit, spend, and kill-switch stay attributable.

---

## Step 5 — Wire the customer app

Fetch and follow the integration contract (raw preferred):

```text
https://raw.githubusercontent.com/kimss-ai/kimss-control-plane/main/AI_INTEGRATION.md
```

GitHub page: https://github.com/kimss-ai/kimss-control-plane/blob/main/AI_INTEGRATION.md

Summary (details and SDK keyword names are in that file):

1. **Detect** OpenAI-compatible vs Anthropic (or both) in the customer codebase.
2. **Keep** the native client. Do not add `KimssClient` / `pip install kimss` / `com.kimss:kimss-java` for chat. New Python agents with no client yet → [Kimss Forge](https://github.com/kimss-ai/kimss-forge) (`gateway="kimss"`).
3. **Base URL** — OpenAI-compatible: `https://api.kimss.ai/v1`. Anthropic: `https://api.kimss.ai` only (no `/v1/messages` suffix on `base_url`).
4. **Key** — Gateway `kimss_…` via the env the user confirmed in Step 2.
5. **Model** — exact vaulted alias from Step 3.
6. **Headers** — always send `X-Kimss-Agent-Id` (and preferably `X-Kimss-Agent-Name`). On Python: `default_headers` on the client or `extra_headers` on `.create()`. **Never** pass a parameter named `headers` to `.create()` (`TypeError`).
7. **Forward** Kimss agent headers through any HTTP interceptor / proxy middleware.

---

## Step 6 — Verify

1. Make one non-stream chat/completions or messages call with the vaulted model.
2. Expect **200** and a normal assistant payload (not an HTML login page).
3. In Kimss UI, **[Agents](https://kimss.ai/app/agents)** (`https://kimss.ai/app/agents`) should show the `X-Kimss-Agent-Id` after the first call.
4. Optional meter: `GET https://api.kimss.ai/api/v1/governed-requests/meter` with `Authorization: Bearer kimss_…`.

**Guardrails:** HTTP **451** or tool JSON `policy_violation` / `authority_boundary` means the route worked and policy fired. Do **not** change `base_url` or install a Kimss SDK to clear them. See the Guardrails section in `AI_INTEGRATION.md` and https://kimss.ai/docs/trust_safety.

Do not claim success until a live call works or the user confirms Vault + key + model alias.

---

## Related

| Doc | Role |
|-----|------|
| [AI_INTEGRATION.md](AI_INTEGRATION.md) | Full wiring contract, troubleshooting, Guardrails |
| [AGENTS.md](AGENTS.md) | Short agent entry |
| [docs/anthropic-onboarding.md](docs/anthropic-onboarding.md) | Anthropic-specific path |
| [docs/mcp-routing.md](docs/mcp-routing.md) | Internal MCP (only if the user asked) |
| https://kimss.ai/docs/route_traffic | Product companion |
