---
name: team-review
description: Independently review a team-workflow run against the acceptance criteria in runs/<run-id>/brief.md, using the reviewer teammate, and record the result in runs/<run-id>/verification.md. Changes nothing in the deliverables.
disable-model-invocation: true
argument-hint: <run-id> [--base <sha>]
---

You are the lead for a review-only pass over an existing run. The arguments are:

$ARGUMENTS

Read `AGENTS.md` first. Open `runs/<run-id>/brief.md` and `plan.md`; if the folder does not
exist, list `runs/` and stop. The base commit is `--base` if given, else the one the brief names.

## 1. Launch the reviewer

The Agent tool with `name: "reviewer"` and `subagent_type: "reviewer"` (if that type is unknown
in this session, `general-purpose` with the body of `.claude/agents/reviewer.md` pasted in, and
say so in the record). The prompt gives the run id, every acceptance criterion verbatim with its
check, every deliverable path, the base commit, and the standing instruction: report only, run
the gates, change nothing, four states only. When the run has a scribe step, the prompt also
names the `docs/` files the scribe changed and any `runs/<run-id>/artifacts/sop.md`, for the
reviewer to check against the diff (its step 6).

## 2. Record

Write `runs/<run-id>/verification.md` from `workflows/templates/acceptance-checklist.md`: one row
per criterion with its state and evidence, the defects table, the gates table with each
command's exit code and last line, and the verdict. Copy the states into `status.json`
`verification`, set `state` to `verifying` (or `done` if everything is VERIFIED and the brief's
deliverables exist), set `updated_at` from `date -u`.

## 3. Report

Tell the asker the verdict, the failed and not-tested criteria with their evidence, and whether
a repair round is needed. Do not fix anything in this skill; a repair is `/team-continue`.
