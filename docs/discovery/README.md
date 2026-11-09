# Discovery

This folder collects workflow frustrations, unmet needs, and ideas
for the product.

Discovery notes are inputs to research and planning. They are not
specifications, implementation tasks, or commitments to build.

## File conventions

- One note per distinct problem or closely related set of observations.
- Name files `YYYY-MM-DD-short-title.md`, using the capture date.
- Use `_template.md` as a starting point.
- Keep filenames stable after creation.
- Link related notes rather than duplicating their contents.

## Capturing notes with an agent

The user may provide rough, incomplete, or conversational notes.
Convert them into readable discovery notes while preserving their intent.

### Process

1. Read existing discovery notes for overlap.
2. Update an existing note if the observation concerns the same need.
   Otherwise, create a new note.
3. Split unrelated observations into separate notes.
4. Use concise, descriptive titles focused on the need or problem.
5. Summarize which files were created or updated.

### Writing rules

- Preserve concrete examples, constraints, and important nuances.
- Separate the observed problem from suggested solutions.
- Preserve explicit preferences such as "must work offline" or
  "I want this exact command"; do not generalize them away.
- Treat user-proposed solutions as ideas unless explicitly stated
  to be requirements or decisions.
- Do not invent motivations, frequency, impact, or requirements.
- Preserve uncertainty: "might be useful" must not become "required."
- Omit optional sections when there is nothing useful to put in them.
- Keep notes proportional to the input; do not pad a small observation.
- Do not create architecture, implementation plans, priorities, or
  acceptance criteria unless requested.
- If an ambiguity materially changes the meaning, ask a focused question.
  Otherwise, record it under "Open questions" and continue.
- Do not change source code or unrelated documentation during capture.

## Revisiting notes

New evidence, recurring incidents, and changed preferences can be added
to existing notes. Preserve meaningful changes of direction rather
than silently rewriting the history.

Research, plans, and ADRs should link back to relevant discovery notes.
A discovery note may link forward to those artifacts when they exist.

