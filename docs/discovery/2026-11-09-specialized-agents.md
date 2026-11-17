# Define specialized agents once and spawn them from a session

Captured: 2026-11-09

## Desired outcome

The user wants to define a specialized agent's behaviour once and for
all, then reuse it. For example, an agent could start a review session
with a specialized agent.

## Ideas

- An `agents/` folder with one file per agent, for example:

  ```
  agents/
  ├── android-ui-tester.md
  ├── pr-creator.md
  └── web-ui-tester.md
  ```

- Spawning one from a running session:

  ```
  mfleet agents spawn web-ui-tester --parent-session=<sessionId>
  ```

## Additional observations

2026-11-17: `--parent-session` shouldn't be needed, since the parent
session can be derived from the env vars the harness sets (see
[Session-aware CLI calls](2026-11-17-session-aware-cli-calls.md)). The
example would become `mfleet agents spawn web-ui-tester`.

## Related

- [Session-aware CLI calls](2026-11-17-session-aware-cli-calls.md)
- [Initial vision](2026-11-09-mfleet-initial-vision.md)
