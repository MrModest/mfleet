# Secrets declared once, unlocked through 1Password

Captured: 2026-11-09

## Desired outcome

All secrets needed by login flows are declared once, and are not exposed
in plain text.

## Ideas

- Declare the secrets in `secrets.sops.yml`.
- Keep the encryption password in 1Password, and use the `op` CLI during
  auth flows.

## Constraints and preferences

- Secrets must not be exposed in plain text.

## Related

- [Credentials break silently inside harnesses](2026-11-09-harness-credentials-break.md)
- [Adding a new host for agents takes manual setup](2026-11-09-new-host-setup.md)
