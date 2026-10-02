---
name: mcp-auth-troubleshooting
description: Use when an MCP tool or local MCP server fails due to authentication, missing environment variables, auth proxy/session-cookie setup, or a newly configured credential not being visible to the running MCP process. Focuses on safe verification without exposing secrets and on fixing process/environment reload issues.
version: 1.0.0
author: Hermes Agent
license: MIT
metadata:
  hermes:
    tags: [mcp, auth, environment, troubleshooting, secrets]
    related_skills: [hermes-agent]
---

# MCP Auth Troubleshooting

## Overview

MCP auth failures often come from the boundary between three environments: the user's shell, the project `.env` file, and the already-running MCP/Hermes process. A user may have "set the token" in one place while the active MCP server still cannot see it.

This skill keeps the loop safe and concrete: call the MCP tool, inspect only presence/shape of configuration (never secret values), identify which runtime cannot see the credential, and give the minimum restart/export step needed.

## When to Use

Use this skill when:

- An MCP tool returns authentication errors such as `401`, `missing token`, `not set`, `unauthorized`, or `refresh token` failures.
- A local MCP server uses a `.env` file, process env vars, session cookies, OAuth refresh tokens, or an auth proxy.
- The user says they configured credentials but the tool still reports them missing.
- You need to verify auth configuration without leaking tokens into chat or logs.

Do not use this skill for:

- Normal API debugging where authentication is already known to work.
- Provider-specific OAuth flows that have a more specific skill.
- Capturing one-off missing-credential state as a durable fact; capture the fix pattern, not the transient failure.

## Safe Troubleshooting Loop

1. Re-run the failing MCP tool first. Completion criterion: you have the current error payload/status from the same path the user cares about.

2. Identify the expected credential source from project docs or config when available. Completion criterion: you know whether the server expects a shell env var, `.env`, config file, keychain, browser cookie, or OAuth cache.

3. Check presence only. Never print secret values. Prefer boolean checks such as:

```bash
if [ -n "${TOKEN_NAME:-}" ]; then echo "TOKEN_NAME is set"; else echo "TOKEN_NAME is NOT set"; fi
```

For `.env` files, check key presence and non-empty value without emitting the value. Completion criterion: you can state which configuration location contains the key and whether it is non-empty.

4. Distinguish shell visibility from process visibility. A token present in the current terminal or `.env` file may still be absent from the already-running MCP server. Completion criterion: you know whether the fix is to edit config, export in the launching environment, or restart the MCP/Hermes process.

5. Give the smallest actionable fix. Examples:

- Add `TOKEN_NAME=...` to the project `.env` file used by the MCP server.
- Export `TOKEN_NAME` before launching the server.
- Restart the MCP/Hermes process after changing env or `.env` so the running server reloads configuration.

6. Verify by re-running the original MCP tool. Completion criterion: the auth error is gone, or the next error is different and accurately reported.

## Secret-Handling Rules

- Do not print, quote, summarize, base64-decode, or copy tokens into the conversation.
- Do not ask the user to paste a token unless there is no other route; prefer telling them where to put it locally.
- When checking files, report only key names and booleans: exists / missing / empty / non-empty.
- Avoid broad environment dumps such as `env`, `printenv`, or `set` because they may expose unrelated secrets.
- If a command accidentally outputs a secret, do not repeat it in the final answer.

## Common Pitfalls

1. Treating `.env` edits as live reloads. Most MCP servers read env at process start. After adding a credential, restart the server or Hermes session that launched it.

2. Checking only the agent shell. MCP tools may run in a different process environment from terminal commands. A shell boolean check can explain the problem, but the original MCP tool is the verification source.

3. Capturing missing credentials as a memory or permanent warning. Missing tokens are setup state. Save the setup/restart pattern in this skill instead.

4. Leaking secrets while trying to be helpful. Presence checks are enough for troubleshooting whether a variable is visible.

5. Stopping after a generic instruction. If tools are available, re-run the MCP tool after the user says they fixed configuration.

## References

- `references/podicom-profile-env.md` — Podicom MCP profile auth example: refresh token expected in `.env` or launch env, and process restart needed after changes.

## Verification Checklist

- [ ] Original MCP tool was re-run, not just reasoned about.
- [ ] Credential source was identified from project docs/config or error payload.
- [ ] Checks reported only presence/non-empty state, not secret values.
- [ ] User was told whether to edit config, export env, or restart the running MCP process.
- [ ] Original MCP tool was used as the final verification path when possible.
