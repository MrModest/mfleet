# One CLI for agent configs, skills, connectors, and agent fleets

Captured: 2026-11-09

## Context

Starting point for `mfleet`. The repo is almost empty: nothing has been
decided and nothing has been built. The user wants to collect wishes and
frustrations with other tools before deciding what `mfleet` should be.

## Desired outcome

A CLI tool, `mfleet`, that:

- handles the user's configs, skills, and connectors;
- also manages and orchestrates fleets of agents.

By the end, it should be working and easy to maintain and improve.

## Ideas

Rough sketch of the config layout (added 2026-11-09):

```
~/.config/mfleet/
├── config.yml
├── connectors/
│   ├── atlassian.yml
│   ├── slack.yml
│   └── github.yml
├── agents/
│   ├── android-ui-tester.md
│   ├── pr-creator.md
│   └── web-ui-tester.md
├── skills/
│   ├── matt-pocock/
│   │   └── wayfinder/
│   │       └── SKILL.md
│   └── delivery-hero/
└── secrets.sops.yml
```

## Constraints and preferences

- (added 2026-11-17) `mfleet` must not depend on any terminal multiplexer
  such as herdr. Agents should be able to use it the same way inside
  herdr, t3-code, the Claude/Codex desktop apps, or a plain Ghostty
  terminal session.
- (added 2026-11-17) The only environment `mfleet` should rely on is the
  harness it runs in: the Claude Code, Codex, and Pi CLIs.

## Open questions

- What should the tool look like? The user doesn't know yet.

## Related

- [Adding a new host for agents takes manual setup](2026-11-09-new-host-setup.md)
- [Credentials break silently inside harnesses](2026-11-09-harness-credentials-break.md)
- [Define specialized agents once and spawn them from a session](2026-11-09-specialized-agents.md)
- [Secrets declared once, unlocked through 1Password](2026-11-09-secrets-via-sops-and-1password.md)
