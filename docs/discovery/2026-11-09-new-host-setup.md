# Adding a new host for agents takes manual setup

Captured: 2026-11-09

## Context

The user sometimes adds a new machine (host) where agents can run.

## Friction

Every new host has to be set up by hand. The user has to make sure that:

- all skills are there;
- MCP servers are authenticated;
- CLI tools are logged in;
- env vars are set.

Some harnesses also fail to log in because of sandbox issues.

## Desired outcome

The user wants to avoid all of that.

## Related

- [Credentials break silently inside harnesses](2026-11-09-harness-credentials-break.md)
- [Secrets declared once, unlocked through 1Password](2026-11-09-secrets-via-sops-and-1password.md)
- [Initial vision](2026-11-09-mfleet-initial-vision.md)
