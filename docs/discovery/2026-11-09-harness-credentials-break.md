# Credentials break silently inside harnesses

Captured: 2026-11-09

## Context

Agents use MCP servers and CLI tools that need credentials. The user
runs several harnesses (Codex is one example).

## Friction

- OAuth tokens expire, and some harnesses don't re-authenticate. The
  user ends up running something like `codex mcp login grafana` by hand
  before an agent can use the MCP tools.
- The user only finds out after asking an agent to do something and
  seeing that it didn't use the expected tool.
- Some CLI tools read auth from an env var. Some harnesses don't export
  that env var in their sandboxed bash, so the tool fails.

## Desired outcome

Agents in any harness can rely on tools being authenticated, without the
user re-logging in by hand or finding out mid-task.

## Ideas

- First idea (since set aside): configure every MCP server and every
  credentialed CLI tool through `mfleet`. The user moved away from this
  because OAuth expires and some harnesses don't re-auth.
- Current idea: wrap these tools as **connectors** and proxy each one
  through `mfleet`. `mfleet` handles the credentials for all harnesses.
- Side benefit: tools that exist only as an MCP server can be used as a
  CLI through the proxy, for example:

  ```
  mfleet connect grafana alerting_rules_read <payload>
  ```

- The proxy should also let agents write code around MCP tool calls,
  similar to Pi's `codemode`.
- Connectors would be declared as files, for example
  `~/.config/mfleet/connectors/grafana.yml` (see the config layout sketch
  in the [initial vision](2026-11-09-mfleet-initial-vision.md)).

## Related

- [Adding a new host for agents takes manual setup](2026-11-09-new-host-setup.md)
- [Secrets declared once, unlocked through 1Password](2026-11-09-secrets-via-sops-and-1password.md)
