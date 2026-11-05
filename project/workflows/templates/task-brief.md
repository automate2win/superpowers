# Brief: <run-id>

Status: DRAFT until the lead has filled every section and the asker has read it.
Written by the lead at intake. The researcher, builder and reviewer read this; none of them
edits it.

## Requested outcome

One paragraph, in the asker's words where possible. What exists at the end that does not exist now.

Team-facing: yes | no (audience: who on the team uses it, or "none"). On yes, the scribe drafts
`runs/<run-id>/artifacts/sop.md` from `workflows/templates/sop.md`.
Size: S | M | L (S: 3 files or fewer, no contract; M: 10 or fewer, no `schema`, `deploy`,
`prod-db` or `coordination` contract; L: anything else). It sets each role's model.

## Inputs and context

- Files, folders, docs and tickets the work starts from (`path`, URL, or "none").
- Repository state the run starts from: branch, base commit (`git rev-parse --short HEAD`).
- Prior runs or decisions that bind this one (`runs/<id>/`, `knowledge/decisions.md` entry).

## Deliverables

Each one a path. A deliverable without a path is not a deliverable.

- `path/to/thing` : what it is.

## Acceptance criteria

Numbered, each one checkable by someone who did not do the work. Say how to check it.

1. AC1: <criterion>. Check: <command, file to open, or thing to observe>.
2. AC2: ...

## Constraints and allowed tools

- Hard rules from `AGENTS.md` that this run touches (read-only upstream, append only, deploy window, no em dashes, audience rules).
- Allowed tools and commands. Anything not listed is not allowed without asking.
- Files or areas that are out of bounds.

## External actions that need approval

Each one listed, with who approves it (the person, in chat). None happens inside the run without that
approval being quoted in `status.json` under `approvals`.

- none / <action>

## Switch rows approved

A copy, for the reader, of the writer switch and kill rows approved for this task. Each is one
line `Switch row: <table> <json>`, with the table and every value, followed by exactly one
line `Switch rollback: <table> <json>`, the row that undoes it, named up front. This section approves
nothing: approval lives only in the registry, as the rows the coordinator passes with
`--switch-row "<table> <json>" --switch-rollback "<table> <json>"` at `bash tools/coord spawn new`
for a spawned task, or at `bash tools/coord task add` (coordinator only) for any other task. A line
here that is not in the registry makes the packet HOLD. The release inserts no row except the
approved ones and, on a red readback, their named rollback rows.

- none

## Assumptions made at intake

Stated so the asker can object. Each one would have been a question; the lead chose an answer.

- <assumption> because <reason>.
