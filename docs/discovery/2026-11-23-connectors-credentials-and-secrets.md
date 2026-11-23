# Connectors, credentials, and secrets

Captured: 2026-11-23

## Context

Agents use MCP servers and CLI tools that need credentials. The user runs
several harnesses (Codex is one example) and sometimes adds a new machine
(host) where agents can run.

## Friction

Credentials break silently:

- OAuth tokens expire, and some harnesses don't re-authenticate. The user
  ends up running something like `codex mcp login grafana` by hand before
  an agent can use the MCP tools.
- The user only finds out after asking an agent to do something and
  seeing that it didn't use the expected tool.
- Some CLI tools read auth from an env var. Some harnesses don't export
  that env var in their sandboxed bash, so the tool fails.

Every new host has to be set up by hand. The user has to make sure that:

- all skills are there;
- MCP servers are authenticated;
- CLI tools are logged in;
- env vars are set.

Some harnesses also fail to log in because of sandbox issues.

## Desired outcome

- Agents in any harness can rely on tools being authenticated, without the
  user logging in again by hand or finding out mid-task.
- Adding a new host doesn't need all that manual setup.
- All secrets that login flows need are declared once and never exposed
  in plain text.
- Agents can search platforms such as eBay, idealo, ImmoScout24, and
  immowelt more efficiently.

## Ideas

Connectors:

- Wrap credentialed tools as **connectors** and proxy each one through
  `mfleet`. `mfleet` handles the credentials for all harnesses.
- Tools that exist only as an MCP server can then be used as a CLI through
  the proxy:

  ```
  mfleet connect grafana alerting_rules_read <payload>
  ```

- The proxy should also let agents write code around MCP tool calls,
  similar to Pi's `codemode`.
- Connectors are declared as files, for example
  `~/.config/mfleet/connectors/grafana.yml` (see the config layout in the
  [overview](2026-11-23-mfleet-overview.md)).
- Connectors for specific platforms, with filtering for searching: eBay,
  idealo, ImmoScout24, immowelt, and so on.
- Set aside: configuring every MCP server and every credentialed CLI tool
  through `mfleet` directly. The user moved away from this because OAuth
  expires and some harnesses don't re-authenticate.

Secrets:

- Declare the secrets in `secrets.sops.yml`.
- Keep the encryption password in 1Password and use the `op` CLI during
  auth flows.

## Constraints and preferences

- Secrets must not be exposed in plain text.

## Related

- [mfleet overview](2026-11-23-mfleet-overview.md), whose doctor section
  notes that agent-fleet's doctor doesn't check credentials
- [Skills](2026-11-23-skills.md), which also need to be on every host
