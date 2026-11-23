# A doctor command

Captured: 2026-11-23

## Context

The user found agent-fleet's `doctor` "might be useful". Reference only,
not a decision.

## Ideas

How [agent-fleet](https://github.com/kirmalyshev/agent-fleet) does it:
`agent-fleet doctor` lists things that need attention, and `--apply`
cleans up what it can. Findings include:

- stale agents (idle, done or blocked for 2+ hours);
- hung agents (working for 30 minutes with low CPU and no change in the
  worktree);
- forgotten worktrees;
- commits left unpushed for over an hour;
- orphan PRs (no live agent on the branch, untouched for 3 days).

It doesn't check credentials, auth expiry, or missing env vars.

## Related

- [Credentials break silently inside harnesses](2026-11-09-harness-credentials-break.md),
  where the user finds out about expired auth only mid-task
