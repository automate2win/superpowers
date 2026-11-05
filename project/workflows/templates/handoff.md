# Handoff: <run-id>

Written by the lead, and only the lead, before stopping. This is a recovery input: the next lead
rebuilds the team from it. It does not restore any agent's context, and saving it does not
preserve any context window.

## Objective

One paragraph. Point at `brief.md` for the full version.

## State at handoff

- Overall: planned / executing / verifying / repairing / blocked / done
- Branch and commit: `<branch>` at `<short sha>` (`git log --oneline -1`)
- Uncommitted changes: yes / no (`git status --short` summary)
- Last gate results: tests rc=<n> "<last line>", typecheck rc=<n> "<last line>"

## Done

- Task ids completed, with the deliverable paths.

## Outstanding

- Task ids not done, with owner, dependency, and what "done" is for each.
- Defects open, by id, with the repair round they are in.

## Blockers and approvals pending

- What is blocked, on whom, and the exact question the asker must answer.

## Key decisions made during the run

- Each one with the reason. Candidates for `knowledge/decisions.md` once reviewed.

## Retro: lesson candidates

Written before `bash tools/coord ready`. Each candidate: what happened, what a later run should
do differently, and the evidence (`file:line`, command, or event id). Candidates stay here; they
are folded into `knowledge/lessons.md` only by a task that already claims `knowledge/`, or by a
`lessons-fold` task once 3 or more are waiting. "None" is a valid answer.

- <candidate>

## Switch rows requested

The writer switch and kill rows this work needs at release, for the coordinator to approve at
spawn, or at task add for a task that was not spawned. Each is one line `Switch row: <table> <json>`,
with the table and every value, followed by exactly one line `Switch rollback: <table> <json>`, the
row that undoes it. A request here approves and inserts nothing; only the registry approves.

- none

## How to continue

1. `/team-continue <run-id>` in a new session, with agent teams enabled.
2. The lead reads this file, `status.json` and `plan.md`, then inspects the actual files and
   `git status` before trusting any of them.
3. Spawn new teammates only for the outstanding tasks. Former teammates are gone.

## Exact next action

One line. The first thing the next lead does.
