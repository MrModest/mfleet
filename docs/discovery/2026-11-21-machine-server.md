# A long-running mfleet server on each machine

Captured: 2026-11-21

## Context

Came up while reviewing [Atlas](https://github.com/bloodfeast/atlas) and
[agent-fleet](https://github.com/kirmalyshev/agent-fleet). Atlas's hooks
mostly call a resident server instead of starting a new process for every
hook event; agent-fleet runs a host process per machine.

## Ideas

- The user thinks `mfleet` needs a long-running server on each machine
  that runs agents.

## Related

- [Agent status without a multiplexer](2026-11-17-agent-status-via-harness-hooks.md)
- [Reduce tool-output tokens in the agent's context](2026-11-17-context-savings-via-hooks.md)
- [Self-update](2026-11-21-self-update.md)
