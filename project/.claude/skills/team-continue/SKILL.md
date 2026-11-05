---
name: team-continue
description: Continue an existing team-workflow run from its saved records under runs/<run-id>/. Reads the handoff, status and plan, checks the real files and git state, then rebuilds the team for the outstanding work. Former teammates are not restored.
disable-model-invocation: true
argument-hint: <run-id, for example 2026-10-25-demo-intake-checklists>
---

You are the new lead of an existing run. The run id is:

$ARGUMENTS

Read `AGENTS.md` first, then `docs/team-workflow-guide.md` if you have not this session.

## 1. Read the record, then distrust it

- Open `runs/<run-id>/handoff.md`, `status.json`, `plan.md`, `brief.md` and
  `verification.md` if present. If the folder does not exist, list `runs/` and stop.
- Then look at what is actually there: `git status --short`, `git log --oneline -5`, the
  deliverable paths the brief names, the checkboxes in `plan.md`. Where the record and the tree
  disagree, the tree wins, and you write the correction into `status.json` with a note.
- Run the gates once for a baseline and record their last lines and exit codes.
- In a coordinated run, read the task's registry state with `bash tools/coord status`. A spawned
  worker resumes its task with its existing claim; it does not run `spawn take` again, since the
  token is spent. A task in `needs_work` resumes at step 5's repair rounds.

## 2. Reconstruct the outstanding work

List every task not `done`, every defect open, every approval `pending`, every criterion not
VERIFIED, whether the scribe step (4b) and its re-check ran, and whether the handoff has its
retro. Confirm the ownership of files is still disjoint for anything you will run in
parallel. A pending approval is still pending: ask the asker, do not infer a yes from the
record.

## 3. Rebuild the team

No former teammate exists. Check which coordination tools this session has (step 0 of the
team-workflow skill) and record it. Then launch only what the outstanding work needs: a builder
per disjoint file set, the reviewer for anything to verify, the scribe for docs not yet updated,
the researcher only if the plan is missing or wrong, each with the model for the task's size. Give each the run id, its tasks, its files and the checks, in the prompt.

## 4. Carry on with steps 3 to 6 of the team-workflow skill

Execution, verification with at most two repair rounds per task, the scribe (4b), the retro and
delivery, and a new handoff before you stop. Set `updated_at` from `date -u` each time you write `status.json`.
