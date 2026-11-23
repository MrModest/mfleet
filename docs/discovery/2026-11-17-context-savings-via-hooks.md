# Reduce tool-output tokens in the agent's context

Captured: 2026-11-17

## Context

The user wants to intercept tool calls (mainly Bash) through harness hooks
to reduce how many tokens tool output puts into the agent's context
window. They pointed at [rtk](https://github.com/rtk-ai/rtk) and
[context-mode](https://github.com/mksglu/context-mode) as examples, and
asked to look at [Atlas](https://github.com/bloodfeast/atlas) for more. The three were researched only through DeepWiki.

Everything under "How others do it" describes those projects. It is a
reference, not a decision on how `mfleet` should work.

## Desired outcome

Less tool-call output in the agent's context, using hooks of the harness
the agent runs in.

## Constraints and preferences

- For things like rtk, the user doesn't want to maintain their own
  version; `mfleet` should use the same rtk under the hood.
- Only the harness environment can be relied on (Claude Code, Codex, Pi);
  see [the vision note](2026-11-09-mfleet-initial-vision.md).
- (added 2026-11-21) An explicit env var to switch off context
  interception, e.g. `MFLEET_CONTEXT_HOOK=0`, so tool output goes through
  as is. For when context processing is half-broken but the rest of the
  CLI should still work. Inspired by Atlas's `ATLAS_RAW=1`, but the user
  wants a more explicit name.

## Ideas

- (added 2026-11-23) Measure the compression percentage by comparing the
  raw output of an MCP tool call or Bash command with the compressed
  output actually sent to the agent. (Atlas instead estimates savings
  against a "what the agent would have read without Atlas" baseline.)

## How others do it

### rtk ([rtk-ai/rtk](https://github.com/rtk-ai/rtk))

Rewrites commands **before** they run (`PreToolUse`), so the output is
compact at the source.

- Rust CLI proxy: `git status` → `rtk git status`. Filters noise, groups
  similar results, shortens diffs, collapses passing tests to a count,
  dedupes repeated lines.
- Integrations: `rtk init -g` (Claude Code hook), `rtk init --codex`
  (Codex `PreToolUse.updatedInput`), plugins or prompt rules for other
  harnesses.
- Interface for another tool: `rtk rewrite "<cmd>"`, reading stdout and
  exit code together:

  | Exit | Meaning |
  |---|---|
  | `0` | Rewritten command printed; allowed by explicit allow rules |
  | `1` | No rewrite; run the original command |
  | `2` | Deny matched; no output = defer to the harness, message = block |
  | `3` | Rewritten command printed, but needs the harness's approval |

  Exit `3` is still a successful rewrite. There are also native hook
  processors (`rtk hook claude`, `rtk hook codex`, ...) that read hook JSON
  on stdin.
- Reported savings are for Bash output only, estimated as bytes / 4.

### context-mode ([mksglu/context-mode](https://github.com/mksglu/context-mode))

An MCP server plus hooks; keeps raw output out of context and lets the
agent search it.

- `ctx_execute` / `ctx_execute_file` run code in a subprocess; only
  printed output enters the context.
- Large output is indexed into a local SQLite FTS5 store; the agent pulls
  relevant snippets with `ctx_search`. `ctx_fetch_and_index` does the same
  for web content.
- Claude Code hooks: `PreToolUse` (Bash, WebFetch, Read, Grep),
  `PostToolUse`, `UserPromptSubmit`, `PreCompact`, `SessionStart`, `Stop`.
  Routing blocks `curl`/`wget`/WebFetch and steers large-output commands to
  the sandbox tools.
- Session state is captured, snapshotted before compaction, and restored
  at session start.
- Codex: hooks via `context-mode hook codex <event>`; `PreCompact` and
  `updatedInput` support depend on the Codex version.

### Atlas ([bloodfeast/atlas](https://github.com/bloodfeast/atlas))

Replaces large output **after** the tool runs (`PostToolUse`). Only Claude
Code is documented.

- Output digests: large Bash/PowerShell/Read/Grep/Glob output is stored in
  full in SQLite; the agent gets a digest in the original tool's output
  shape plus a handle (`h:ID`). Digests are task-specific (test totals and
  failures, compiler diagnostics, grep hits grouped by file, file
  outlines). Details come back via `recall key=h:ID` with selectors
  (`item`, `failures`, `lines`, `grep`), capped at 8 KB per slice. Small
  output passes through; errors fail open; `ATLAS_RAW=1` bypasses it.
- Light `PreToolUse`: doesn't wrap commands (that would change what the
  harness asks permission for); only appends a read-only exit-code tail so
  failing commands still reach `PostToolUse`.
- Repeated identical reads return "unchanged since your call #N" or only
  the changed lines; `fresh: true` forces the full answer.
- Memory recall on prompt submit and subagent start, gated by relevance
  and a budget (600 tokens by default); never injects the same record
  twice per session.
- 14 tools listed instead of 47; four core tools always loaded, the rest
  on demand.
- Compaction: `PreCompact` writes a checkpoint (active work, files, recent
  tool results, pending work); after compaction `SessionStart` injects it
  and checks the summary against it. "Misses" count the agent looking up
  the same unchanged target again after compaction.
- Measurement: `--token-report` estimates savings against a "without
  Atlas" baseline, with a `Net` view that subtracts Atlas's own context
  cost; `--hook-report` / `--tune-floor` check whether injected memories
  were used and suggest a relevance threshold.

### agent-fleet

Does nothing for context savings. Its `hook bash` is only a safety guard
(refuses raw `kill`/`pkill`/`killall`), and its hooks fail open with a
one-second cap.

## Additional observations

2026-11-17: rtk and Atlas act at different points and could be combined:
rtk rewrites before the command runs (needs the harness to support
rewriting tool input), Atlas digests whatever is still large afterwards
(needs the harness to support replacing tool output).

## Related

- [Agent status without a multiplexer](2026-11-17-agent-status-via-harness-hooks.md),
  which also proposes `mfleet hooks`
- [Code index for repos agents work in](2026-11-17-code-index.md)
- [Event log idea, borrowed from agent-fleet](2026-11-17-event-log-from-agent-fleet.md)
