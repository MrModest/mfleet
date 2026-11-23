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

## Constraints and preferences

- (added 2026-11-21) Failure behaviour should be configurable per hook.
  Most hooks can fail without stopping the agent, but some of the user's
  hooks may be deal-breakers: if those fail, the agent should hard stop.
  (agent-fleet instead always fails open: its hook exits successfully on
  any error and gives up after one second.)
- (added 2026-11-23) Safe killing of processes can be part of the hooks
  flow. In agent-fleet, a Bash hook refuses raw `kill` / `pkill` /
  `killall` from agents, and its own kill command never kills harness
  processes or the caller's parent processes, re-checks that a process is
  really hung, then sends TERM followed by KILL.
- (added 2026-11-23) After auto-compaction, the hooks flow should
  re-inject crucial behavioural skills or prompts, e.g. "you are an
  orchestrator session, ..." or "you are a UI tester, ...". The
  compaction summary already keeps track of the work itself, so Atlas's
  checkpoint/resume-card approach isn't needed.

## Additional observations

2026-11-21: what each harness gives a hook (from Context7 docs; for
reference):

| | Claude Code | Codex | Pi |
|---|---|---|---|
| Mechanism | shell command, JSON on stdin | shell command, JSON on stdin (`hooks.json`) | TypeScript extension, `pi.on("<event>", handler)` |
| Common fields | `session_id`, `transcript_path`, `cwd`, `permission_mode`, `prompt_id`; `agent_id` / `agent_type` inside subagents | `session_id`, `turn_id`, `transcript_path`, `cwd`, `model`, `permission_mode`, `agent_id` / `agent_type` | `ctx.sessionManager` (session state up to the current message) |
| Tool events | `tool_name`, `tool_input`, `tool_use_id` (+ the tool's response after it runs) | `tool_name`, `tool_input`, `tool_use_id` | `tool_call`: `toolName`, `toolCallId`, mutable `input`; `tool_result`: the result, patchable |
| Other | `UserPromptSubmit` has `prompt` and `source` (`user`, `sdk`, `system`, ...) | `SessionStart` has `source` (`startup`, `resume`, `clear`, `compact`, `fork`); `Stop` has `last_assistant_message` | |
| Block / stop | exit 2 blocks the tool call and shows stderr to the agent; exit 1 is a non-blocking error shown only to the user; JSON output can also stop the session | similar JSON output | return `{ block: true, reason, terminate }`; `terminate` stops the agent |
| Change output | `PostToolUse` can return `updatedToolOutput` and `additionalContext` | `PreToolUse.updatedInput` (version-dependent) | `tool_result` handlers can replace the content |

Pi has no shell-command hooks, so hooking into Pi means a small Pi
extension.

## Related

- [Event log idea, borrowed from agent-fleet](2026-11-17-event-log-from-agent-fleet.md)
- [Reduce tool-output tokens in the agent's context](2026-11-17-context-savings-via-hooks.md),
  another use for harness hooks
- [Session-aware CLI calls](2026-11-17-session-aware-cli-calls.md)
- [Define specialized agents once and spawn them from a session](2026-11-09-specialized-agents.md)
