# Skills managed or served through mfleet

Captured: 2026-11-17

## Context

The user's current setup: skills live in `~/.agents/skills`, and each
skill folder is automatically symlinked into `~/.claude/skills`.

The user doesn't fully follow the
[home repo pattern](https://www.aihero.dev/the-home-repo-pattern), which
recommends keeping skills in the home repo instead of the user folder.

## Ideas

- A `mfleet skills` subcommand that agents use to find and read skills,
  instead of the harness's built-in skill discovery:
  - `mfleet skills search 'write for human'`
  - `mfleet skills list`

  The user might consider fully delegating skills to it, but hasn't
  decided.
- Alternatively, `mfleet skills` as a plain skill manager:
  - many skills are collected in `~/.config/mfleet/skills`;
  - `mfleet skills install humanise` symlinks the skill into the
    `.agents` and `.claude` folders in the home dir (global);
  - `mfleet skills install humanise --project` puts it into the
    respective project folders instead.

## Related

- [One CLI for agent configs, skills, connectors, and agent fleets](2026-11-09-mfleet-initial-vision.md),
  which sketches `~/.config/mfleet/skills/`
- [Adding a new host for agents takes manual setup](2026-11-09-new-host-setup.md)
- [Code index for repos agents work in](2026-11-17-code-index.md)
