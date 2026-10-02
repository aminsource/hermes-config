# Podicom MCP profile auth example

## Scenario

A user asked for the Podicom profile via the `mcp_podicom_profile` tool. The tool returned a 401 payload saying `PODICOM_REFRESH_TOKEN is not set`. After the user said they had set it, re-running the same MCP tool still returned the same error.

Safe local checks showed:

- The current agent shell did not have `PODICOM_REFRESH_TOKEN` set.
- The project `.env` file existed, but did not contain a `PODICOM_REFRESH_TOKEN` entry.
- No token value was printed.

## Durable pattern

For Podicom MCP, the auth proxy expects `PODICOM_REFRESH_TOKEN` from the launch environment or project `.env`. If the value is added after the MCP process is already running, restart the MCP/Hermes process so the proxy/server reloads its environment.

Use the original MCP tool as the verification path after restart. If the missing-token error disappears, continue with the next returned status rather than assuming the whole auth flow is complete.

## Safe check pattern

When inspecting `.env`, report only key presence and whether it is non-empty. Do not print the value.
