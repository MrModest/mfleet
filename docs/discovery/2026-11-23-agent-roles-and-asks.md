# Agent roles and who can ask whom

Captured: 2026-11-23

## Context

Came up while looking at
[agent-fleet](https://github.com/kirmalyshev/agent-fleet), which has agent
roles (implementer, lead, root coordinator) and lets agents message each
other and ask the human questions.

## Desired outcome

Agents can ask each other questions, but only within limits set by their
role.

## Ideas

- A command for asking another agent, e.g.
  `mfleet agents ask --agent-id=<agent-id>`.
- Roles as a property of agents: orchestrator, implementor, verifier,
  reviewer. Each role defines what the agent can do.

## Constraints and preferences

Examples given by the user:

- A verifier or reviewer can't ask questions of any random agent, only of
  its parent session.
- An orchestrator can ask questions of all its related sub-sessions, but
  shouldn't touch other orchestrators, especially ones not related to the
  same project.

## Related

- [Define specialized agents once and spawn them from a session](2026-11-09-specialized-agents.md)
- [Session-aware CLI calls](2026-11-17-session-aware-cli-calls.md), which
  knows the calling session and its parent
- [Tasks subcommand for agents](2026-11-17-agent-tasks-subcommand.md)
