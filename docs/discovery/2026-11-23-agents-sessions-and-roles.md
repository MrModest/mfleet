# Agents, sessions, and roles

Captured: 2026-11-23

## Desired outcome

- The user defines a specialized agent's behaviour once and reuses it.
  For example, an agent starts a review session with a specialized
  agent.
- An orchestrator session delegates work to other agents and is promptly
  notified when a delegated agent has finished its job.
- The CLI knows which session each request comes from, so audit works
  across all subcommands.
- Agents can ask each other questions, within limits set by their role.

## Constraints and preferences

- `mfleet` runs normal harness sessions as sub-agents. The harnesses'
  built-in sub-agents aren't used at all, which avoids their quirks and
  limitations.
- The orchestrator session decides which harness and which model a
  delegated agent runs. The agent file sets a default model, and the
  orchestrator can override it. So rule-based routing (as in agent-fleet,
  which picks a profile from an issue's title or labels) isn't needed.
- Session detection relies only on the harness (Claude Code, Codex, Pi),
  not on multiplexer env vars.

## Agent type files

One file per agent type, in an `agents/` folder:

```
~/.config/mfleet/agents/
├── android-ui-tester.md
├── pr-creator.md
└── web-ui-tester.md
```

Each file is Markdown with frontmatter. (YAML files were considered too.)

- The frontmatter sets a default model for the agent type. The spawning
  orchestrator can override it. Example: the default is `gpt-6.1-terra`,
  but its limits have run out, so the orchestrator spawns the agent with
  `sonnet-5.5` instead.
- The Markdown body is the agent's system prompt or brief, written as a
  template with placeholders instead of plain text. Placeholders include
  the agent name, the task, and the parent session ID.

The template idea comes from agent-fleet's `brief.md`, which uses
placeholders like `@@NAME@@`, `@@TASK@@`, and `@@BRANCH@@`, filled in at
spawn. agent-fleet refuses to spawn if a placeholder has no value.
`mfleet`'s placeholder syntax isn't decided.

## Spawning and lifecycle

Spawning from a running session:

```
mfleet agents spawn web-ui-tester
```

The parent session comes from the env vars the harness sets (see
"Knowing the calling session" below), so `--parent-session=<sessionId>`
isn't needed.

The user believes something like agent-fleet's spawn flow could be part
of `mfleet agents`. agent-fleet creates a fresh git worktree, runs an
optional setup script, launches the harness, sends the first prompt, and
cleans up on failure.

For this, a lifecycle for delegated agents needs to be defined, so the
main session is promptly notified when a delegated agent is done.

An orchestrator can wait on a delegated agent in the background:

```
mfleet agents <agent-id> wait
```

Waiting on anything else, such as GitHub CI, isn't needed. Agents can use
existing tools for that. (This came up from
[Atlas](https://github.com/bloodfeast/atlas), where queued work can wait
on a PR review and a GitHub webhook marks it done.)

## Roles and who can ask whom

Roles are a property of an agent, separate from its agent type:

- orchestrator;
- implementor;
- verifier;
- reviewer.

Each role defines what the agent can do. (agent-fleet has the roles
implementer, lead, and root coordinator, and lets agents message each
other and ask the human questions.)

A command for asking another agent:

```
mfleet agents ask --agent-id=<agent-id>
```

The user's examples of limits:

- A verifier or reviewer can't ask questions of any random agent, only of
  its parent session.
- An orchestrator can ask questions of all its related sub-sessions, but
  shouldn't touch other orchestrators, especially ones not related to the
  same project.

## Knowing the calling session

Harnesses set env vars in the shell where the agent runs commands, so
`mfleet` can recognize the session implicitly. Agents don't have to pass
a session ID on every call. What the user observed:

| Harness | Session ID env var | Notes |
|---|---|---|
| Codex | `CODEX_SESSION_ID`, `CODEX_THREAD_ID` (same value in the example) | Also `CODEX_VERSION` and others |
| Claude Code | `CLAUDE_CODE_SESSION_ID` | Also `CLAUDECODE=1`, `CLAUDE_CODE_CHILD_SESSION`, `CLAUDE_PID`, `CLAUDE_CODE_MESSAGING_SOCKET`, and a messaging token var |
| Pi | `PI_SESSION_ID` | Only set when the agent runs a command, not for `!` commands typed by the user. Also `PI_SESSION_FILE`, `PI_MODEL`, `PI_PROVIDER` |

Other env vars the user saw, which `mfleet` must not depend on:

- herdr: `HC_SCOPE`, `HC_TERMINAL_ID`, `HC_TERMINAL_SOCKET` (a unix
  socket path).
- T3 Code: `T3CODE_CLI_PATH` (path to the `t3` CLI, e.g.
  `~/.t3/bin/t3`) and `T3CODE_THREAD_ID` (a UUID).

Fallback if implicit detection doesn't work out: a mandatory
`--current-session-id=UUID` argument on every CLI call.

Tasks don't use the session to limit visibility. The session ID only
records who created a task (see
[Tasks shared between agents and humans](2026-11-23-tasks.md)).

## Related

- [mfleet overview](2026-11-23-mfleet-overview.md)
- [Harness hooks](2026-11-23-harness-hooks.md), which tracks agent status
- [Event log, as agent-fleet does it](2026-11-23-event-log-reference.md),
  with its spawn and message events
