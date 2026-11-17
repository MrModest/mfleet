# Session-aware CLI calls

Captured: 2026-11-17

## Desired outcome

The CLI knows which session each request comes from, so that:

- each session sees only the things related to it;
- audit works.

This applies to all subcommands, not only tasks.

## Ideas

- If nothing better is found, a mandatory `--current-session-id=UUID`
  argument on every CLI call.
- Implicit session detection from env vars that harnesses already set in
  the agent's shell, so agents don't have to pass the argument on every
  call (see "Additional observations").

## Open questions

- Is there a better way than a mandatory argument? The user said "if we
  can't find anything better", so the argument is a fallback, not a
  decision.

## Additional observations

2026-11-17: harnesses set env vars in the shell where the agent runs
commands. The user observed:

| Harness | Session ID env var | Notes |
|---|---|---|
| Codex | `CODEX_SESSION_ID`, `CODEX_THREAD_ID` (same value in the example) | Also `CODEX_VERSION` and others |
| Claude Code | `CLAUDE_CODE_SESSION_ID` | Also `CLAUDECODE=1`, `CLAUDE_CODE_CHILD_SESSION`, `CLAUDE_PID`, `CLAUDE_CODE_MESSAGING_SOCKET` and a messaging token var |
| Pi | `PI_SESSION_ID` | Only set when the agent runs a command, not for `!` commands typed by the user. Also `PI_SESSION_FILE`, `PI_MODEL`, `PI_PROVIDER` |

Agents run inside the herdr app also get these env vars:

- `HC_SCOPE`
- `HC_TERMINAL_ID`
- `HC_TERMINAL_SOCKET` (a unix socket path)

The user did not say yet what these could be used for.

The user's takeaway: `mfleet` can recognize the session implicitly from
these, so agents don't need to pass `--current-session-id` every time.

## Related

- [Tasks subcommand for agents](2026-11-17-agent-tasks-subcommand.md)
- [Define specialized agents once and spawn them from a session](2026-11-09-specialized-agents.md),
  which already passes a session ID via `--parent-session=<sessionId>`
- [Credentials break silently inside harnesses](2026-11-09-harness-credentials-break.md),
  where connector calls go through the same CLI
