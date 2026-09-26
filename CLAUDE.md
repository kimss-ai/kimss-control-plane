# Claude Code / Claude

This repo is the public Kimss control-plane hub.

If the user asked you to **onboard Kimss** (or only shared this repository), follow **[ONBOARDING.md](ONBOARDING.md)** in order — welcome first, then Gateway key, vaulted model, agent id, wire, verify. Do not invent keys or model aliases.

Wiring contract: **[AI_INTEGRATION.md](AI_INTEGRATION.md)**. Short rules: **[AGENTS.md](AGENTS.md)**.

Do not clone this repo into the customer app. Keep the native OpenAI or Anthropic SDK; use a Gateway `kimss_...` key; always send `X-Kimss-Agent-Id`.
