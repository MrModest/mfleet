# How mfleet config is layered

Captured: 2026-11-21

## Context

Came up while reviewing [Atlas](https://github.com/bloodfeast/atlas) and
[agent-fleet](https://github.com/kirmalyshev/agent-fleet).

- Both treat unknown or misspelled config keys as errors, keep config in
  the standard user config folder (`~/.config/<tool>` on Linux), and let
  an env var point to another folder. The user agrees with this.
- In agent-fleet, a repo's own config replaces the machine config entirely
  rather than merging with it.

## Ideas

- A hierarchy with overriding: global → machine → repo. The user isn't
  sure where harness-specific config fits in it.
- Alternatively, profiles (as in agent-fleet, where a profile names a
  harness, its arguments, env vars to remove, etc.) that can be combined
  and merged, instead of a fixed hierarchy. The user doesn't know yet how
  it would look.

## Related

- [One CLI for agent configs, skills, connectors, and agent fleets](2026-11-09-mfleet-initial-vision.md)
- [Define specialized agents once and spawn them from a session](2026-11-09-specialized-agents.md)
