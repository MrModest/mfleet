# Tasks subcommand for agents

Captured: 2026-11-17

## Desired outcome

Agents can add, update, delete, and check tasks through the CLI.

## Ideas

- A `mfleet tasks` subcommand.
- (added 2026-11-17) Task shape sketch:

  ```ts
  type TaskCreator = {
    host: string;
    harness: "codex" | "claude" | "pi";
    sessionId: UUID;
  };

  type Task = {
    createdBy: TaskCreator;
    parent?: TaskId;        // parent task
    children: TaskId[];     // child tasks
    status: "new" | "in-progress" | "blocked" | "done" | "cancelled";
    blockedBy: TaskId[];    // other tasks' IDs
  };
  ```

## Additional observations

2026-11-17: the user changed their mind on session scoping. Tasks should
not be bound to the session that created them, so agents can freely pick
them up and work on them in parallel. The current session ID is only used
to record who created the task (`createdBy`). This came up after looking
at agent-fleet, where tasks are fleet-wide too (see
[Event log idea, borrowed from agent-fleet](2026-11-17-event-log-from-agent-fleet.md)).

## Related

- [Session-aware CLI calls](2026-11-17-session-aware-cli-calls.md), which
  provides the session ID for `createdBy`
- [Event log idea, borrowed from agent-fleet](2026-11-17-event-log-from-agent-fleet.md)
