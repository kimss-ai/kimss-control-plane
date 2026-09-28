# Guided onboarding — Kimss gateway

**Agent-agnostic workflow** for Cascade, Cursor, Claude Code, Codex, Windsurf, Devin, and any other coding assistant.

**Fetch URL (prefer this over cloning):**

```text
https://raw.githubusercontent.com/kimss-ai/kimss-control-plane/main/ONBOARDING.md
```

When a user says something like “onboard Kimss” and points at this repo (or only pastes the repo link), **run this workflow in order**. Do **not** invent a Gateway key, vaulted model id, or agent slug. Pause after each step that needs a user answer.

Do **not** clone this repo into the customer app. Rewire the **customer** codebase only. Wiring contract: [`AI_INTEGRATION.md`](https://raw.githubusercontent.com/kimss-ai/kimss-control-plane/main/AI_INTEGRATION.md).

---

## Step 1 — Ask for the Gateway API key (say this first)

Keep every reply in this step to **a few short sentences**. Do **not** explain Kimss architecture, base URL, vault aliases, headers, SDKs, or what will change yet — that comes after the key is confirmed in an env var.

Ask only:

1. Do they already have a Gateway API key (`kimss_…` — not an OpenAI/Anthropic provider key)?
2. Which env var holds it (or should hold it): `KIMSS_API_KEY`, `OPENAI_API_KEY`, or `ANTHROPIC_API_KEY`?
3. Optionally a short key prefix (e.g. `kimss_…waYU`) so you can confirm the right key — **never** ask them to paste the full key.

### If they do not have a key yet

Reply with **only** this, then wait:

> Mint a Gateway key at https://kimss.ai/app/keys, put it in `KIMSS_API_KEY` (or tell me which env var to use), and say when it is set. Do not paste the full key here.

### If they minted a key but have not set an env var yet

Reply with **only** this, then wait:

> Put that `kimss_…` key in `KIMSS_API_KEY` (or name the env var you prefer). Confirm when it is set — do not paste the full key.

**Stop and wait** for confirmation that a key is available via an env var before any other onboarding talk or file edits. Then continue to Step 2.

---

## Step 2 — Confirm the Gateway API key

Once they reply:

1. Accept the env var name they chose (or default to `KIMSS_API_KEY`).
2. Treat “I minted it and put it in `KIMSS_API_KEY`” (or an equivalent prefix confirmation) as enough — do **not** re-ask for the full secret.
3. **Never** paste the full key into chat. **Never** commit it.

If they already confirmed a key (or prefix) and env var, move on to Step 3. If they only minted and still need to place it, use the short “put it in env” reply from Step 1 and wait again.

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
