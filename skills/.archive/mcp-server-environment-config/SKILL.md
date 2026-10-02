---
name: mcp-server-environment-config
description: "Configure and troubleshoot environment variables for Hermes-managed MCP servers."
version: 1.0.0
author: Hermes Agent
license: MIT
metadata:
  hermes:
    tags: [hermes, mcp, environment, configuration, troubleshooting]
---

# MCP Server Environment Configuration

Use this skill when a user asks to add API keys, backend URLs, auth tokens, or other environment variables for a Hermes-managed MCP server, or when an MCP tool reports that an env var is missing even after the user says they set it.

If the task is specifically about Hermes Agent, also load the protected `hermes-agent` skill first and treat official Hermes docs/CLI output as authoritative. This skill captures the practical MCP env workflow without modifying protected bundled/hub skills.

## Core workflow

1. Determine the active Hermes env file, do not guess:
   - Run `hermes config env-path`.
   - For the default profile this is usually `~/.hermes/.env`; profile-specific sessions may use a profile-specific env path.

2. Check the MCP server registration:
   - Run `hermes mcp list` to confirm the server name and transport.
   - Run `hermes mcp test <name>` after edits to verify the server starts and tools are discovered.

3. Put MCP runtime secrets in the Hermes env file, not only the project repo `.env`:
   - Project `.env` may be useful for local `npm run dev` or direct repo commands.
   - Hermes-launched MCP servers inherit from the Hermes process/env, so the Hermes env file is the durable place for MCP server credentials.

4. Preserve existing env content safely:
   - Update or append only the requested keys.
   - Do not print secret values back to the user.
   - Set restrictive permissions (`0600`) on env files that contain secrets.

5. Reload/restart after editing:
   - Existing MCP tool instances may keep the old environment.
   - Tell the user to run `/reload-mcp` or restart Hermes if a tool still reports a missing env var.
   - If the CLI supports it, `/reload` reloads env variables for the session, while `/reload-mcp` reloads MCP servers.

6. Verify at the right level:
   - `hermes mcp test <name>` only proves connection/tool discovery.
   - A real MCP tool call proves the backend credentials are accepted.
   - If discovery succeeds but the backend returns 401/403, the env variable is present but the credential may be stale, malformed, or for the wrong auth mechanism.

## Auth-token pitfalls

- Do not assume similarly named variables are interchangeable. Inspect the MCP server code or docs if a tool reports a missing variable; e.g. a server may read `PODICOM_REFRESH_TOKEN` even if the user provided `PODICOM_AUTHORIZATION`.
- If a provided token is a wrapper/JWT that embeds another token, only extract an inner value when the server code clearly expects that inner value and the user asked you to use the provided env material. Avoid exposing the extracted secret in responses.
- A backend response like `token.not.active` means the variable was transmitted but rejected by the upstream service. Capture the fix as "use a fresh token/cookie from login", not as "the MCP tool is broken".

## Safe response pattern

When finished, report only:

- Which env path was updated.
- Which variable names are present (not their values).
- What verification command/tool was run and what class of result it produced.
- Whether a reload/restart is needed.

## Session notes

See `references/podicom-mcp-env.md` for a concrete Podicom MCP example involving `PODICOM_BASE_URL`, `PODICOM_REFRESH_TOKEN`, `hermes config env-path`, `hermes mcp test podicom`, and `/reload-mcp`.