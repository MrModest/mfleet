# Harness hooks: agent status, safety, and context savings

Captured: 2026-11-23

## Context

`mfleet` can only rely on the harness it runs in (see the
[overview](2026-11-23-mfleet-overview.md)). Harness hooks are how it
would see what agents do. Three uses came up: tracking agent status,
keeping agents safe and on-role, and cutting how many tokens tool output
puts into the context.

## Friction

agent-fleet gets agent status from herdr, which `mfleet` must not depend
on. Registration there is push-based through herdr, not through harness
hooks. The host process subscribes to herdr's socket events and turns
them into presence events in the log (from the user's paste, citing
agent-fleet's `PROTOCOL.md` and `host.rs`):

| herdr event | agent-fleet event |
|---|---|
| `pane.agent_detected` | `agent.up` |
| `pane.closed`, `pane.exited`, `workspace.closed` | `agent.down` |
| `pane.agent_status_changed` (per agent) | `agent.state` |

On startup the host calls herdr's `agent.list` to emit `agent.up` for
everything already running, and marks `agent.down` anything the hub
thought was up but herdr no longer has. Each state change is published to
the hub (or the local queue), so the state stays current without any
cooperation from the agent program.

This only works inside herdr. `mfleet` should work in herdr, T3 Code, the
Claude or Codex desktop apps, or a plain terminal.

## Desired outcome

- `mfleet` knows agent status (started, stopped, failed, and so on) using
  only the harness: Claude Code, Codex, or Pi.
- Less tool-call output (mainly Bash) in the agent's context.

## Ideas

- Maybe a `mfleet hooks` subcommand, registered in each harness's settings
  for session start, stop, failure, and similar events.
- Measure the compression percentage by comparing the raw output of an MCP
  tool call or Bash command with the compressed output actually sent to
  the agent. (Atlas instead estimates savings against a "what the agent
  would have read without Atlas" baseline.)

## Constraints and preferences

- Failure behaviour is configurable per hook. Most hooks can fail without
  stopping the agent. Some of the user's hooks may be deal-breakers: if
  those fail, the agent should hard stop. (agent-fleet always fails open:
  its hook exits successfully on any error and gives up after one second.)
- Safe killing of processes can be part of the hooks flow. In agent-fleet,
  a Bash hook refuses raw `kill`, `pkill`, and `killall` from agents. Its
  own kill command never kills harness processes or the caller's parent
  processes, re-checks that a process is really hung, then sends TERM
  followed by KILL.
- After auto-compaction, the hooks flow re-injects important behavioural
  skills or prompts, such as "you are an orchestrator session, ..." or
  "you are a UI tester, ...". The compaction summary already keeps track
  of the work itself, so Atlas's checkpoint and resume-card approach isn't
  needed.
- For tools like rtk, the user doesn't want to maintain their own version.
  `mfleet` uses the same rtk under the hood.
- An explicit env var switches off context interception, e.g.
  `MFLEET_CONTEXT_HOOK=0`, so tool output goes through as is. This is for
  when context processing is half-broken but the rest of the CLI should
  still work. Inspired by Atlas's `ATLAS_RAW=1`, with a more explicit name.

## What each harness gives a hook

From Context7 docs, for reference:

| | Claude Code | Codex | Pi |
|---|---|---|---|
| Mechanism | shell command, JSON on stdin | shell command, JSON on stdin (`hooks.json`) | TypeScript extension, `pi.on("<event>", handler)` |
| Common fields | `session_id`, `transcript_path`, `cwd`, `permission_mode`, `prompt_id`; `agent_id` and `agent_type` inside subagents | `session_id`, `turn_id`, `transcript_path`, `cwd`, `model`, `permission_mode`, `agent_id`, `agent_type` | `ctx.sessionManager` (session state up to the current message) |
| Tool events | `tool_name`, `tool_input`, `tool_use_id`, plus the tool's response after it runs | `tool_name`, `tool_input`, `tool_use_id` | `tool_call`: `toolName`, `toolCallId`, mutable `input`; `tool_result`: the result, patchable |
| Other | `UserPromptSubmit` has `prompt` and `source` (`user`, `sdk`, `system`, ...) | `SessionStart` has `source` (`startup`, `resume`, `clear`, `compact`, `fork`); `Stop` has `last_assistant_message` | |
| Block or stop | exit 2 blocks the tool call and shows stderr to the agent; exit 1 is a non-blocking error shown only to the user; JSON output can also stop the session | similar JSON output | return `{ block: true, reason, terminate }`; `terminate` stops the agent |
| Change output | `PostToolUse` can return `updatedToolOutput` and `additionalContext` | `PreToolUse.updatedInput` (depends on the version) | `tool_result` handlers can replace the content |

Pi has no shell-command hooks, so hooking into Pi means a small Pi
extension.

## How others save context

Reference only, not a decision. Researched through DeepWiki.

### rtk ([rtk-ai/rtk](https://github.com/rtk-ai/rtk))

Rewrites commands **before** they run (`PreToolUse`), so the output is
compact at the source.

- A Rust CLI proxy: `git status` becomes `rtk git status`. It filters
  noise, groups similar results, shortens diffs, collapses passing tests
  to a count, and dedupes repeated lines.
- Integrations: `rtk init -g` (Claude Code hook), `rtk init --codex`
  (Codex `PreToolUse.updatedInput`), and plugins or prompt rules for other
  harnesses.
- Interface for another tool: `rtk rewrite "<cmd>"`. The caller reads
  stdout and the exit code together:

  | Exit | Meaning |
  |---|---|
  | `0` | Rewritten command printed; allowed by explicit allow rules |
  | `1` | No rewrite; run the original command |
  | `2` | Deny matched; no output means defer to the harness, a message means block |
  | `3` | Rewritten command printed, but needs the harness's approval |

  Exit `3` is still a successful rewrite. There are also native hook
  processors (`rtk hook claude`, `rtk hook codex`, ...) that read hook
  JSON on stdin.
- Reported savings cover Bash output only, estimated as bytes / 4.

### context-mode ([mksglu/context-mode](https://github.com/mksglu/context-mode))

An MCP server plus hooks. It keeps raw output out of the context and lets
the agent search it.

- `ctx_execute` and `ctx_execute_file` run code in a subprocess. Only
  printed output enters the context.
- Large output is indexed into a local SQLite FTS5 store, and the agent
  pulls relevant snippets with `ctx_search`. `ctx_fetch_and_index` does
  the same for web content.
- Claude Code hooks: `PreToolUse` (Bash, WebFetch, Read, Grep),
  `PostToolUse`, `UserPromptSubmit`, `PreCompact`, `SessionStart`, `Stop`.
  Routing blocks `curl`, `wget`, and WebFetch, and steers large-output
  commands to the sandbox tools.
- Session state is captured, snapshotted before compaction, and restored
  at session start.
- Codex: hooks via `context-mode hook codex <event>`. `PreCompact` and
  `updatedInput` support depend on the Codex version.

### Atlas ([bloodfeast/atlas](https://github.com/bloodfeast/atlas))

Replaces large output **after** the tool runs (`PostToolUse`). Only
Claude Code is documented.

- Output digests: large Bash, PowerShell, Read, Grep, and Glob output is
  stored in full in SQLite. The agent gets a digest in the original
  tool's output shape plus a handle (`h:ID`). Digests are specific to the
  kind of output: test totals and failures, compiler diagnostics, grep
  hits grouped by file, file outlines. Details come back via
  `recall key=h:ID` with selectors (`item`, `failures`, `lines`, `grep`),
  capped at 8 KB per slice. Small output passes through, errors fail
  open, and `ATLAS_RAW=1` bypasses it.
- A light `PreToolUse`: it doesn't wrap commands, because that would
  change what the harness asks permission for. It only appends a
  read-only exit-code tail so failing commands still reach `PostToolUse`.
- Repeated identical reads return "unchanged since your call #N" or only
  the changed lines. `fresh: true` forces the full answer.
- Memory recall on prompt submit and subagent start, gated by relevance
  and a budget (600 tokens by default). The same record is never injected
  twice per session.
- 14 tools listed instead of 47. Four core tools are always loaded, the
  rest on demand.
- Compaction: `PreCompact` writes a checkpoint (active work, files,
  recent tool results, pending work). After compaction, `SessionStart`
  injects it and checks the summary against it. "Misses" count the agent
  looking up the same unchanged target again after compaction.
- Measurement: `--token-report` estimates savings against a "without
  Atlas" baseline, with a `Net` view that subtracts Atlas's own context
  cost. `--hook-report` and `--tune-floor` check whether injected memories
  were used and suggest a relevance threshold.

### agent-fleet

Does nothing for context savings. Its `hook bash` is only a safety guard
(it refuses raw `kill`, `pkill`, and `killall`), and its hooks fail open
with a one-second cap.

### rtk and Atlas together

rtk and Atlas act at different points and could be combined. rtk
rewrites before the command runs, which needs the harness to support
rewriting tool input. Atlas digests whatever is still large afterwards,
which needs the harness to support replacing tool output.

## Related

- [mfleet overview](2026-11-23-mfleet-overview.md), with the per-machine
  server idea that hooks could call
- [Agents, sessions, and roles](2026-11-23-agents-sessions-and-roles.md)
- [Event log, as agent-fleet does it](2026-11-23-event-log-reference.md)
- [Code index](2026-11-23-code-index.md)
