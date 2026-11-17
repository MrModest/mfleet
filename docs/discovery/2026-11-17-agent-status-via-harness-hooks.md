# Agent status without a multiplexer

Captured: 2026-11-17

## Context

`mfleet` needs to know which agents are running and what state they are
in. agent-fleet gets this from herdr, which `mfleet` must not depend on
(see [the vision note](2026-11-09-mfleet-initial-vision.md)).

How agent-fleet does it (from the user's paste, citing its `PROTOCOL.md`
and `host.rs`):

- Registration is push-based through herdr, not through harness hooks. The
  host daemon subscribes to herdr's socket events and turns them into
  presence events in the log:

  | herdr event | agent-fleet event |
  |---|---|
  | `pane.agent_detected` | `agent.up` |
  | `pane.closed`, `pane.exited`, `workspace.closed` | `agent.down` |
  | `pane.agent_status_changed` (per agent) | `agent.state` |

- On startup the host calls herdr's `agent.list` to emit `agent.up` for
  everything already running, and marks `agent.down` anything the hub
  thought was up but herdr no longer has.
- Each state change is published to the hub (or the local spool), so the
  DB stays current without any cooperation from the agent program.

## Friction

agent-fleet's approach only works inside herdr. `mfleet` should work in
herdr, T3 Code, the Claude/Codex desktop apps, or a plain terminal, so it
needs its own way to track agent status.

## Desired outcome

`mfleet` knows agent status (started, stopped, failed, etc.) using only
the harness it runs in (Claude Code, Codex, Pi).

## Ideas

- A `mfleet hooks` subcommand, registered in each harness's settings for
  session start, stop, failure, and similar hook events. Not a decision;
  the user said "maybe".

## Related

- [Event log idea, borrowed from agent-fleet](2026-11-17-event-log-from-agent-fleet.md)
- [Session-aware CLI calls](2026-11-17-session-aware-cli-calls.md)
- [Define specialized agents once and spawn them from a session](2026-11-09-specialized-agents.md)
