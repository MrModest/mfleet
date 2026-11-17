# Event log idea, borrowed from agent-fleet

Captured: 2026-11-17

## Context

The user asked to look at
[kirmalyshev/agent-fleet](https://github.com/kirmalyshev/agent-fleet)
(researched only through DeepWiki) for things worth borrowing for
`mfleet`. The user asked to capture its event log approach as a note.

This is how agent-fleet does it. It is a reference, not a decision on how
`mfleet` should work. agent-fleet is built around herdr (identity, presence
and message delivery all go through it); the user doesn't want `mfleet` to
depend on any multiplexer (see
[the vision note](2026-11-09-mfleet-initial-vision.md)).

## Ideas

How agent-fleet does it:

- Every fact is a typed event in an append-only log (SQLite on a central
  hub). Current state is not stored; it is derived by replaying events in
  order through a pure reducer (`State::fold`).
- Common envelope on every event:
  - `id`: [ULID](https://github.com/ulid/spec) chosen by the writer, so a duplicate append does nothing
    and replay is safe;
  - `v`: schema version of the event type;
  - `ts`, `actor` (`host` / `agent` / `human`), `project`;
  - optional `task` and `re` (reference to an earlier event's `id`);
  - `type` plus type-specific `data`.
- `re` links follow-ups to their cause: `spawn.result` → `spawn.requested`,
  `msg.delivered` → `msg.sent`, `ask.answered` → `ask.raised`.
- Event type groups: agent presence/state, tasks and dependencies,
  blockers, asks to the human, messages, spawn, stuck/signals, processes
  and worktrees, resources, self-update.
- Writes are strict (unknown types or bad fields are rejected); reads are
  tolerant (unknown types are kept in the log and skipped by the reducer).
- When the hub is unreachable, ordinary writes queue locally and replay
  later; exclusive task claims refuse instead.

Example events (field names come from agent-fleet's protocol docs; the
values are made up to look like real data, not copied from a real log):

```jsonc
// operator "me" spawns an agent for issue acme/webapp#123
{ "id": "01KA3F9Q2V7XG5M8T4N6R1BZCD", "v": 1, "ts": "2026-11-17T10:42:05Z",
  "actor": { "kind": "agent", "id": "agent:laptop/main/me" },
  "project": "webapp", "task": null, "re": null,
  "type": "spawn.requested",
  "data": { "name": "claude-123", "agent": "agent:laptop/main/claude-123",
            "host": "host:laptop", "session": "main", "repo": "webapp",
            "branch": "fix/123-login-redirect", "profile": "claude",
            "brief": "Fix login redirect loop (acme/webapp#123) ...",
            "why": "issue acme/webapp#123" } }

// the host started it; `re` points to the request
{ "id": "01KA3F9VZ0E2M6PB8C1D4HJKQR", "v": 1, "ts": "2026-11-17T10:42:19Z",
  "actor": { "kind": "host", "id": "host:laptop" },
  "project": "webapp", "task": null, "re": "01KA3F9Q2V7XG5M8T4N6R1BZCD",
  "type": "spawn.result",
  "data": { "agent": "agent:laptop/main/claude-123", "ok": true } }

// the spawned agent's task moves to working
{ "id": "01KA3FA2KQ5N9W3Y7T0V8S6XGM", "v": 1, "ts": "2026-11-17T10:42:31Z",
  "actor": { "kind": "agent", "id": "agent:laptop/main/claude-123" },
  "project": "webapp", "task": "task:webapp/123", "re": null,
  "type": "task.state", "data": { "state": "working" } }

// operator sends it a message
{ "id": "01KA3FH8C4R7T2X9B5N1M6Q0WE", "v": 1, "ts": "2026-11-17T10:58:47Z",
  "actor": { "kind": "agent", "id": "agent:laptop/main/me" },
  "project": "webapp", "task": "task:webapp/123", "re": null,
  "type": "msg.sent",
  "data": { "to": "agent:laptop/main/claude-123", "kind": "INFO",
            "ref": "task:webapp/123", "text": "also check the logout flow" } }

// delivered once the host saw the agent start working on it
{ "id": "01KA3FHB6T1V5Z8D2K4P9R3NYA", "v": 1, "ts": "2026-11-17T10:59:02Z",
  "actor": { "kind": "host", "id": "host:laptop" },
  "project": "webapp", "task": null, "re": "01KA3FH8C4R7T2X9B5N1M6Q0WE",
  "type": "msg.delivered",
  "data": { "to": "agent:laptop/main/claude-123", "observed": "working" } }
```

ID formats (human-readable names, not UUIDs):

- `agent:<host>/<session>/<name>`: `<host>` is the fleet host name
  (defaults to the machine hostname), `<session>` is the herdr session
  name, `<name>` is the agent's name in herdr (falls back to the pane ID).
  Spawned agents are named profile prefix + issue number, e.g.
  `claude-123`. The README's own example is `agent:laptop/main/me`.
- `host:<name>`: a machine running the host agent, e.g. `host:laptop`.
- `task:<repo name>/<issue number or name>`: e.g. `acme/webapp#123`
  becomes `task:webapp/123` (repo name only, no owner); work without an
  issue gets a name, e.g. `task:default/build-cache`. Task IDs are
  fleet-wide, not scoped to a session; ownership is a separate field
  pointing to an agent ID.
- `human:board`: the human answering through the web board.
- Event `id`: a [ULID](https://github.com/ulid/spec) (26 chars, time-sortable), referenced by `re`.

## Related

- [Session-aware CLI calls](2026-11-17-session-aware-cli-calls.md),
  which wants audit across all subcommands
- [Tasks subcommand for agents](2026-11-17-agent-tasks-subcommand.md)
- [Define specialized agents once and spawn them from a session](2026-11-09-specialized-agents.md)
