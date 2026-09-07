# Contributing

## Repo scope (customer-facing)

This repository is the **public hub** for buyers, investors, security reviewers, and integrators. Keep it aligned with that audience.

**Belongs here**

- OpenAPI control-plane contract, examples, and conformance tests
- Integrator guides (Anthropic, A2A / `AI_INTEGRATION.md`)
- Buyer / decision-maker capability brief
- Trust signals (SECURITY, Scorecard / Best Practices badges, LICENSE)

**Does not belong here**

- Internal GitHub org / account migration playbooks
- Repo-metadata sync scripts, social-preview render/upload tooling, Playwright setup helpers
- Personal or historical assignment repos, credential notes, or operator runbooks

Internal ops for public GitHub hygiene live in `kimssApi` (`scripts/kimss_public_github/`) and the `kimss-docs` vault — not in this tree.

## Source of truth

The live Kimss API is implemented in **`kimssApi`** (`kimss-ai/kimssApi`, `src/app.py` + `kimssapi_functions/`). This repo is a **public contract mirror** — not the runtime.

When you change control-plane routes or request/response shapes in kimssApi:

1. Update `openapi/control-plane.yaml` in the same change window.
2. Extend `conformance/test_spec_grounding.py` if you add paths or required fields.
3. Bump `info.version` in the OpenAPI file.

## Verify locally

```bash
pip install pyyaml pytest
pytest conformance/
```

## Do not

- Invent endpoints that are not deployed at `https://api.kimss.ai`.
- Publish decrypted MCP credentials or real API keys in examples.
