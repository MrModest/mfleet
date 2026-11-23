# mfleet overview: one CLI for agent configs, skills, connectors, and fleets

Captured: 2026-11-23

## Context

`mfleet` is a CLI the user wants to build. Nothing has been decided or
built yet. The user is collecting wishes, frustrations with other tools,
and ideas from other projects before deciding what `mfleet` should be.

This note is the entry point. Each area has its own note, linked below.

## Desired outcome

A CLI tool, `mfleet`, that:

- handles the user's configs, skills, and connectors;
- manages and orchestrates fleets of agents.

In the end it should work, and be easy to maintain and improve.

## Constraints and preferences

- `mfleet` must not depend on any terminal multiplexer, such as herdr.
  Agents should be able to use it the same way inside herdr, T3 Code, the
  Claude or Codex desktop apps, or a plain Ghostty terminal session.
- The only environment `mfleet` relies on is the harness it runs in: the
  Claude Code, Codex, and Pi CLIs.
- Ideas borrowed from other projects are references, not decisions.

## Areas

| Area | Subcommands mentioned so far | Note |
|---|---|---|
| Agents, sessions, roles | `mfleet agents spawn`, `mfleet agents wait <agent-id>`, `mfleet agents ask <agent-id>` | [Agents, sessions, and roles](2026-11-23-agents-sessions-and-roles.md) |
| Tasks | `mfleet tasks` | [Tasks shared between agents and humans](2026-11-23-tasks.md) |
| Harness hooks: status, safety, context savings | `mfleet hooks` (maybe) | [Harness hooks](2026-11-23-harness-hooks.md) |
| Connectors, credentials, secrets, new hosts | `mfleet connectors` | [Connectors, credentials, and secrets](2026-11-23-connectors-credentials-and-secrets.md) |
| Skills | `mfleet skills` | [Skills](2026-11-23-skills.md) |
| Code index | `mfleet index` | [Code index](2026-11-23-code-index.md) |
| Event log (reference) | | [Event log, as agent-fleet does it](2026-11-23-event-log-reference.md) |
| Doctor | `mfleet doctor` | this note |

All subcommand names are ideas.

## Config

Rough sketch of the config layout:

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

Agent files are Markdown with frontmatter (see
[Agents, sessions, and roles](2026-11-23-agents-sessions-and-roles.md)).

The user agrees with what Atlas and agent-fleet both do:

- unknown or misspelled config keys are errors;
- config lives in the standard user config folder (`~/.config/<tool>` on
  Linux);
- an env var can point to another config folder.

How config is layered is not decided. Two ideas:

- A fixed hierarchy where each level overrides the previous one: global,
  then machine, then repo. The user isn't sure where harness-specific
  config fits in it. (In agent-fleet, a repo's own config replaces the
  machine config entirely instead of merging with it.)
- Profiles that can be combined and merged, instead of a fixed hierarchy.
  In agent-fleet a profile names a harness, its arguments, env vars to
  remove, and so on. The user doesn't know yet how this would look.

## A long-running server on each machine

The user thinks `mfleet` needs a long-running server on each machine that
runs agents. This came from Atlas, whose hooks mostly call a resident
server instead of starting a new process for every hook event, and from
agent-fleet, which runs a host process per machine.

## Self-update

The user thinks self-update "might be useful". Reference only.

How Atlas does it:

- Release builds check for updates at session start, at most once an
  hour, with a 10-second limit. When offline or on error, the installed
  version starts anyway (fails open).
- Trust chain: a manifest signed with Ed25519 is checked against a public
  key built into the binary. Then the downloaded binary's SHA-256 is
  checked against the manifest.
- An update can't replace a newer version with an older one. A version
  can be pinned, and updates and notices can be switched off.
- A running server downloads the new binary but keeps serving the old one
  until it is reconnected.

agent-fleet instead updates from source: each machine pulls the git clone
its binary was built from, rebuilds, and restarts.

## Doctor

The user thinks a doctor command "might be useful". Reference only.

How agent-fleet does it: `agent-fleet doctor` lists things that need
attention, and `--apply` cleans up what it can. Findings include:

- stale agents (idle, done, or blocked for 2+ hours);
- hung agents (working for 30 minutes with low CPU and no change in the
  worktree);
- forgotten worktrees;
- commits left unpushed for over an hour;
- orphan PRs (no live agent on the branch, untouched for 3 days).

It doesn't check credentials, auth expiry, or missing env vars, which is
where the user's own pain is (see
[Connectors, credentials, and secrets](2026-11-23-connectors-credentials-and-secrets.md)).

## Prior art

These projects were researched through DeepWiki only. What they do is
recorded as reference in the notes, not as decisions.

- [agent-fleet](https://github.com/kirmalyshev/agent-fleet): Rust. A
  central hub, a host process per machine, and herdr for identity,
  presence, and message delivery. The user doesn't want the herdr part.
- [Atlas](https://github.com/bloodfeast/atlas): a Rust MCP server with
  hooks, documented for Claude Code only. Output digests, code index,
  memory recall, compaction checkpoints, signed self-update.
- [rtk](https://github.com/rtk-ai/rtk): rewrites shell commands so their
  output is compact.
- [context-mode](https://github.com/mksglu/context-mode): an MCP server
  plus hooks that keeps raw output out of the context and makes it
  searchable.
- [The home repo pattern](https://www.aihero.dev/the-home-repo-pattern):
  one repo holds skills, `AGENTS.md`, and shared docs, and agents start
  from it. The user follows it partly.

## Open questions

- What should the tool look like overall? The user doesn't know yet.
- How config is layered: fixed hierarchy or profiles, and where
  harness-specific config goes.
