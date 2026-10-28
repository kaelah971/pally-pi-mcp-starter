# Troubleshooting

Short, tested steps for the Pally × Pi MCP Starter (Pi 1.3.0).

## Authentication steps that worked during testing

1. Ensure project config exists with only the public endpoint:

   ```bash
   cat .pi/mcp.json
   # {"mcpServers":{"pally":{"url":"https://agent.pally.com/mcp"}}}
   ```

   Or create it:

   ```bash
   cp mcp.example.json .pi/mcp.json
   # or: pi mcp add -l pally --url https://agent.pally.com/mcp
   ```

2. Approve project trust when Pi prompts (project config is ignored until trusted).

3. Sign in:

   ```bash
   pi mcp login pally
   ```

   Complete the browser flow (default timeout 300s).

4. Verify — no Pally call needed:

   ```bash
   pi mcp list
   # expect: pally: connected, 3 tools (codemode, project)
   ```

   ```bash
   pi mcp list --json
   # expect pally entry: scope project, exposure codemode,
   # transport https://agent.pally.com/mcp, state connected
   ```

5. In-session alternative: run `/mcp` to inspect, reconnect, or sign in; run `/reload` after editing config outside the session.

## Redirect URI mismatch (observed once)

- **Observed:** one initial OAuth attempt reported a redirect URI mismatch before a later `pi mcp login pally` succeeded.
- Exact URIs/ports are omitted to avoid leaking identifiers.
- This starter does **not** claim Pally caused the mismatch, nor that `pi mcp login pally` is a universal fix. A single observation cannot prove a Pally bug, a Pi bug, or the only workaround. Possible loopback-callback handling is an unproven hypothesis.
- If you hit it: retry `pi mcp login pally` in a fresh shell, then re-run `pi mcp list`. Check `~/.pi/agent/mcp.log` for the connection error (redact tokens before sharing).

## Common misconfigurations

- **Wrong file:** project config must be `.pi/mcp.json` (project root), not the user-level file. Confirm `scope: project` in `pi mcp list --json`.
- **Trust not granted:** re-launch Pi and approve project trust.
- **Wrong transport:** Pi rejects `sse`. Use `https://agent.pally.com/mcp`, not an `/sse` variant.
- **Secrets in config:** remove any `headers` / `bearer-token-env-var` / `oauth` blocks before committing. This starter is URL-only by design.

## Get help without leaking secrets

- Share `pi --version`, `pi mcp list` state line, and redacted `pi mcp list --json` (name/scope/exposure/state only).
- Never share `mcp-auth.json`, OAuth codes, tokens, email bodies, or contact details.
