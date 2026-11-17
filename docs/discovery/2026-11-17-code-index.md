# Code index for repos agents work in

Captured: 2026-11-17

## Context

Came up while looking at the code index of
[Atlas](https://github.com/bloodfeast/atlas) (researched through
DeepWiki).
The user follows the
[home repo pattern](https://www.aihero.dev/the-home-repo-pattern): one
repo (e.g. `acme/home`) holds the skills, `AGENTS.md` and shared docs, and
agents are started from it. Its `AGENTS.md` has a pointer per repo, like
`Storefront (Next.js app): ~/repos/acme/storefront. Clone from <remote>
if missing.` The agent works in that clone (or clones it there first),
usually in a per-feature git worktree. Code doesn't move into the home
repo.

The user doesn't follow every recommendation of the pattern (e.g. where
skills live; see
[Skills managed or served through mfleet](2026-11-17-skills-via-mfleet.md)).

The user wanted to know whether the index follows the repo the file
belongs to, not the home repo the agent was opened in.

## Desired outcome

Agents get code-index help for whichever repo they actually read or grep
in, even when the session was started somewhere else.

## Ideas

- A `mfleet index` subcommand:
  - `mfleet index run .` indexes the current repo;
  - `mfleet index list` shows the indexed repos;
  - and so on.

## Additional observations

2026-11-17: how Atlas ([bloodfeast/atlas](https://github.com/bloodfeast/atlas))
does it. Reference only, not a decision:

- Grep/Read annotations add the enclosing function and its line range to
  matches, e.g.
  `456: pub fn post_tool_use(...) [fn Router::post_tool_use 456-...]`, and
  a note naming functions cut off by a partial Read.
- Index lookup follows the target file's path, not the session's folder.
  Absolute paths work, and so does `cd DIR && grep ...` within one Bash
  command. Only recognized read commands are annotated (`grep -n`,
  `rg -n`, line-range prints).
- Repos must be registered in advance as code roots (`code.roots` in
  `atlas.json`). Files outside every code root get nothing; there is no
  documented automatic or lazy indexing of a repo the agent wanders into.
- The session's folder still picks the active project (memory store and
  work queue), falling back to `default_project`. Separately,
  `active_repo` tracks the repo the session touched most recently, which
  recall uses for ranking.
- Not documented: whether repos listed in `atlas.project.json`
  (`repos[]`) also become code roots, which root wins when roots overlap,
  and how pipes, scripts or `git -C` are handled.
- Indexed roots are re-indexed in the background when files change.

## Related

- [Reduce tool-output tokens in the agent's context](2026-11-17-context-savings-via-hooks.md)
