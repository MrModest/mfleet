# Code index

Captured: 2026-11-23

## Context

The user follows the
[home repo pattern](https://www.aihero.dev/the-home-repo-pattern), but
not every recommendation of it (for example, where skills live; see
[Skills](2026-11-23-skills.md)). In the pattern, one repo (e.g.
`acme/home`) holds the skills, `AGENTS.md`, and shared docs, and agents
are started from it. Its `AGENTS.md` has a pointer per repo, like:

```
Storefront (Next.js app): ~/repos/acme/storefront. Clone from <remote> if missing.
```

The agent works in that clone (or clones it there first), usually in a
per-feature git worktree. Code doesn't move into the home repo.

So an agent often reads and greps in a repo other than the one its
session started in. The user wanted to know whether a code index follows
the repo a file belongs to, not the home repo the agent was opened in.

## Desired outcome

Agents get code-index help for whichever repo they actually read or grep
in, even when the session started somewhere else.

## Ideas

A `mfleet index` subcommand:

- `mfleet index run .` indexes the current repo;
- `mfleet index list` shows the indexed repos;
- and so on.

## How Atlas does it

[Atlas](https://github.com/bloodfeast/atlas), researched through
DeepWiki. Reference only.

- Grep and Read annotations add the enclosing function and its line range
  to matches, e.g.
  `456: pub fn post_tool_use(...) [fn Router::post_tool_use 456-...]`,
  plus a note naming functions cut off by a partial Read.
- Index lookup follows the target file's path, not the session's folder.
  Absolute paths work, and so does `cd DIR && grep ...` within one Bash
  command. Only recognized read commands are annotated (`grep -n`,
  `rg -n`, line-range prints).
- Repos must be registered in advance as code roots (`code.roots` in
  `atlas.json`). Files outside every code root get nothing. There is no
  documented automatic or lazy indexing of a repo the agent wanders into.
- The session's folder still picks the active project (memory store and
  work queue), falling back to `default_project`. Separately,
  `active_repo` tracks the repo the session touched most recently, which
  recall uses for ranking.
- Indexed roots are re-indexed in the background when files change.
- Not documented: whether repos listed in `atlas.project.json`
  (`repos[]`) also become code roots, which root wins when roots overlap,
  and how pipes, scripts, or `git -C` are handled.

## Related

- [Harness hooks](2026-11-23-harness-hooks.md), where Atlas's other
  hook-based features are described
