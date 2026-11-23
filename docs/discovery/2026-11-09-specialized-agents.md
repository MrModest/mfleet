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

2026-11-23: after looking at how
[agent-fleet](https://github.com/kirmalyshev/agent-fleet) spawns agents
(fresh git worktree, optional setup script, launch the harness, send the
first prompt, clean up on failure), the user believes something like that
could be part of the `mfleet agents` flow. For that, a lifecycle for
delegated agents needs to be defined, so the main session is promptly
notified when a delegated agent has finished its job.

2026-11-23: an orchestrator agent could wait on a delegated agent in the
background with something like `mfleet agents <agent-id> wait`. Waiting
on anything else (e.g. GitHub CI) isn't needed: agents can use existing
tools for that. (Came up from
[Atlas](https://github.com/bloodfeast/atlas), where queued work can wait
on a PR review and a GitHub webhook marks it done.)

2026-11-23: agent types are already covered by the `agents/` definitions
above, so agent-fleet's roles (implementer, lead, coordinator) aren't
needed. (Later the same day the user did want roles, as a separate
property that defines what each agent may do; see
[Agent roles and who can ask whom](2026-11-23-agent-roles-and-asks.md).)

2026-11-23: the user wants one file per agent type declaration: YAML, or
a Markdown file with frontmatter (as the `.md` files in the earlier
sketches above and in the vision note could be). Both could work; not
decided. Later the same day the user chose Markdown with frontmatter.
The frontmatter defines a default model for the agent type, and the
spawning orchestrator session can override it when needed. Example: the
default is `gpt-6.1-terra`, but its limits have run out, so the
orchestrator spawns it with `sonnet-5.5` instead. The Markdown body is
the agent's system prompt / brief, written as a template with the
placeholders above (agent name, task, parent session ID, ...). The
agent's system prompt / brief could be a template with placeholders
instead of plain text, like agent-fleet's `brief.md` (`@@NAME@@`,
`@@TASK@@`, `@@BRANCH@@`, ... filled in at spawn; spawning refuses if a
placeholder has no value). The brief can contain the agent name, the
task, and the parent session ID.

2026-11-23: the orchestrator session decides which harness and which
model each delegated agent runs (refined later: the agent file sets a
default model, the orchestrator can override it), so rule-based routing (as in
agent-fleet, which picks a profile from an issue's title or labels) isn't
needed.

2026-11-23: one point of `mfleet` is to run normal sessions as
sub-agents, so the harnesses' built-in sub-agents aren't used at all,
avoiding their quirks and limitations.

## Related

- [Session-aware CLI calls](2026-11-17-session-aware-cli-calls.md)
- [Initial vision](2026-11-09-mfleet-initial-vision.md)
