# Claude Code / Claude

This repo is the public Kimss control-plane hub.

If the user asked you to **onboard Kimss** (or only shared this repository), follow **[ONBOARDING.md](ONBOARDING.md)** in order — first reply is only the mint link at `/app/keys` plus put the key in `KIMSS_API_KEY`. If they minted a key but have not set an env var yet, tell them only to put it in `KIMSS_API_KEY` and wait. Then vaulted model, agent id, wire, verify. Do not invent keys or model aliases.

Wiring contract: **[AI_INTEGRATION.md](AI_INTEGRATION.md)**. Short rules: **[AGENTS.md](AGENTS.md)**.

Do not clone this repo into the customer app. Keep the native OpenAI or Anthropic SDK; use a Gateway `kimss_...` key; always send `X-Kimss-Agent-Id`.
