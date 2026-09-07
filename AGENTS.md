# Agent instructions (Cascade / Cursor / Claude Code)

This repository is the **public Kimss control-plane hub** for buyers and integrators.

**To wire a customer application to the Kimss gateway, fetch and follow:**

https://raw.githubusercontent.com/kimss-ai/kimss-control-plane/main/AI_INTEGRATION.md

Do **not** clone this repo into the customer app. Keep the native OpenAI or Anthropic SDK; point `base_url` at `api.kimss.ai`; use a Gateway `kimss_...` key; always send `X-Kimss-Agent-Id`.

Companion product doc: https://kimss.ai/docs/route_traffic
