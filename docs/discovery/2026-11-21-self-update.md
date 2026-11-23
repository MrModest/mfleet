# Self-update

Captured: 2026-11-21

## Context

The user found this "might be useful" while reviewing how
[Atlas](https://github.com/bloodfeast/atlas) updates itself. Reference
only, not a decision.

## Ideas

How Atlas does it:

- Release builds check for updates at session start, at most once an
  hour, with a 10-second limit. Offline or on error, the installed version
  just starts (fails open).
- Trust chain: a manifest signed with Ed25519 is checked against a public
  key built into the binary, then the downloaded binary's SHA-256 is
  checked against the manifest.
- Can't replace a newer version with an older one; a version can be
  pinned; updates and notices can be switched off.
- A running server downloads the new binary but keeps serving the old one
  until it is reconnected.

For comparison, [agent-fleet](https://github.com/kirmalyshev/agent-fleet)
updates from source: each machine pulls the git clone its binary was built
from, rebuilds, and restarts.

## Related

- [A long-running mfleet server on each machine](2026-11-21-machine-server.md)
