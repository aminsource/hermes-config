# Podicom Hermes MCP env configuration lesson

This reference captures a reusable setup pattern discovered while configuring the Podicom MCP server for Hermes.

## What happened

The Podicom MCP server was configured in `/home/hooman/.hermes/config.yaml` as a stdio server:

```yaml
mcp_servers:
  podicom:
    command: bash
    args:
      - -lc
      - cd /home/hooman/Projects/hamin/ai/podicom/podicom-mcp && exec node dist/index.js
    enabled: true
```

Putting `PODICOM_BASE_URL` and `PODICOM_REFRESH_TOKEN` only in `.env` files did not affect the already-running MCP subprocess. The correct durable location for this setup is the per-server MCP env block:

```yaml
mcp_servers:
  podicom:
    command: bash
    args:
      - -lc
      - cd /home/hooman/Projects/hamin/ai/podicom/podicom-mcp && exec node dist/index.js
    env:
      PODICOM_BASE_URL: https://podicom.ir:8080
      PODICOM_REFRESH_TOKEN: <user-supplied token>
    enabled: true
```

## Why this matters

Hermes filters the environment passed to stdio MCP subprocesses. Secrets and service-specific env vars should be explicitly listed under `mcp_servers.<name>.env` if the MCP server needs them.

## Verification pattern

- Verify config structure without printing secrets.
- Report token presence and length only.
- Run `hermes mcp test podicom` if terminal tools are available.
- The user can run `/reload-mcp`; after reload, direct MCP tools should work.

## Cleanup pattern

If the user asks to remove duplicate env locations, remove Podicom keys from:

- `/home/hooman/.hermes/.env`
- project `.env` files such as `/home/hooman/Projects/hamin/ai/podicom/podicom-mcp/.env`

Keep the canonical values in `/home/hooman/.hermes/config.yaml` under `mcp_servers.podicom.env`.

## Podicom tool usage observed

- `mcp_podicom_profile` with `action: get` returns merchant profile data.
- `mcp_podicom_shops` with `action: list`, `page: 0`, `size: 20` returns a paginated shop list.
