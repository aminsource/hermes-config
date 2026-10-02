# Podicom MCP env configuration example

This note captures a concrete troubleshooting pattern for a Podicom MCP server launched by Hermes.

## Symptoms

- User says they set Podicom env variables.
- `mcp_podicom_profile` returns an error equivalent to `PODICOM_REFRESH_TOKEN is not set`.
- Project repo `.env` may be present, but the currently running Hermes MCP process does not see the variable.

## Correct target for Hermes-launched MCP servers

Use `hermes config env-path` to locate the active Hermes env file. In the observed default-profile case, it returned:

`/home/hooman/.hermes/.env`

Update that file with the MCP server variables, not just the project `.env`.

For Podicom, the relevant variable names are:

- `PODICOM_BASE_URL`
- `PODICOM_REFRESH_TOKEN`

The Podicom MCP server code reads `PODICOM_REFRESH_TOKEN`; a user-provided `PODICOM_AUTHORIZATION` variable is not equivalent unless the server is changed to read it.

## Verification sequence

1. Add/update only the requested keys in the Hermes env file.
2. Ensure the env file is mode `0600` when it contains secrets.
3. Run `hermes mcp test podicom`.
   - Success here means the MCP server connects and tools are discovered.
   - It does not prove the upstream Podicom credential is accepted.
4. If an already-running MCP tool still reports a missing env var, tell the user to run `/reload-mcp` or restart Hermes.
5. Retry a real tool call, such as the profile `get` action.

## Backend auth distinction

If the server sees the env var but Podicom returns HTTP 401 with a code like `token.not.active` / Persian message "توکن نامعتبر است", the variable reached the backend but the credential is invalid or stale. Ask for a fresh `rt` cookie/refresh token from a logged-in browser session rather than treating it as an MCP configuration failure.

## Privacy

Do not print JWTs, refresh tokens, cookies, or extracted inner token values. Report variable names, presence, length if needed, and verification status only.