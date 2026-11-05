---
name: team-workflow
description: Start a team-workflow run from an outcome. The lead does intake, plans with the researcher teammate, builds with the builder teammate(s), verifies with the reviewer teammate, documents with the scribe teammate, and keeps the durable run records under runs/<run-id>/. Also holds the release session's procedure. Guide in docs/team-workflow-guide.md.
disable-model-invocation: true
argument-hint: <the outcome you want, in one or two sentences>
---

You are the lead of a small team for one run. The outcome requested is:

$ARGUMENTS

Read `docs/team-workflow-guide.md` once if you have not this session, and `AGENTS.md` first in
any case. Then run the steps below in order. You own intake, scope, acceptance criteria,
dependencies, approvals, integration and delivery; you are the only writer of `brief.md`,
`status.json`, `verification.md` and `handoff.md`. Teammates get task-specific context in their
prompts; they do not see this conversation.

## 0. Check what this session can do

Record in the run, before anything else:

- Whether you are a spawned worker. If the prompt that started this session says
  `bash tools/coord spawn take <token>`, run exactly that first and do what it prints: it aligns
  this worktree to `coord/integration`, claims the task and prints the brief, whose run folder
  replaces step 1's intake. If `bash tools/coord version` does not print `registry schema 2`,
  the spawn commands are not installed: say so and work as an ordinary session.
- Which coordination tools you have: the Agent tool with a `name` parameter, `SendMessage`,
  `ListAgents`, and whether `TaskCreate` / `TaskList` / `TaskUpdate` exist (load them with
  ToolSearch `select:TaskCreate,TaskList,TaskUpdate`; if they do not load, the shared task list
  is not available here and you coordinate by prompt and `SendMessage`, recording tasks in
  `status.json` only).
- Whether `.claude/agents/researcher.md`, `builder.md` and `reviewer.md` are available as agent
  types for the Agent tool. If a type is unknown in this session, use `general-purpose` and
  paste the role file's body into the prompt, and record that the role was applied by prompt.
  The same for `scribe.md`.
- Whether the Agent tool takes a per-call `model`. If it does, step 3 sets each role's model by
  size; if it does not, the role files' defaults apply, and you record that.

Never say a team was created unless a teammate was actually launched and answered.

## 1. Intake

- Run id: `$(date -u +%Y-%m-%d)-<slug>`. Create `runs/<run-id>/` and `runs/<run-id>/artifacts/`.
- Copy `workflows/templates/task-brief.md` to `runs/<run-id>/brief.md` and fill every section:
  outcome, inputs and base commit, deliverables as paths, numbered acceptance criteria each with
  its check, constraints and allowed tools, external actions needing approval, assumptions, and
  the `Team-facing:` and `Size:` lines.
- Ask the asker only what blocks safe execution. Everything else is an assumption, written down.
- Write the first `status.json` (shape below) with `state: "intake"`.

## 2. Planning

Launch the researcher: the Agent tool with `name: "researcher"`, `subagent_type: "researcher"`,
and a prompt that gives the run id, the brief's path, the deliverables and the acceptance
criteria in full, and says: return the plan as text, write nothing. Save its plan to
`runs/<run-id>/plan.md`. Check that every task has an owner, the files it owns, and its
dependency and size; tasks that run in parallel must own disjoint files. Parallelise only
genuinely independent work; one builder is the default. When the plan ends with a `coord-tasks`
block, run `bash tools/coord split-check runs/<run-id>/plan.md; echo "rc=$?"`: exit 1 is an
overlap and exit 3 a missing contract cover, and either sends the plan back to the researcher
before anyone claims anything.

Stop and ask the asker before proceeding if the plan expands scope, needs a paid service, a
sensitive integration, a deploy, a push, a migration applied, or any destructive change.
Record the question and the answer in `status.json` under `approvals`.

Update `status.json`: `state: "planned"`, the tasks with ids, owners, deps and `files`.

## 3. Execution

For each builder: the Agent tool with `name: "builder"` (or `builder-2`), `subagent_type:
"builder"`, and a prompt that gives the run id, the task ids it owns, the exact files or
directories it may touch, the checks to run, and whether to commit (default: no). Where the
Agent tool takes a `model`, pass the one for the task's size:

| Size | Researcher | Builder | Reviewer | Scribe |
| --- | --- | --- | --- | --- |
| S | sonnet | sonnet | opus | haiku |
| M | opus | sonnet | opus | sonnet |
| L | opus | opus | opus | sonnet |

The reviewer is `opus` at every size. Two builders
never share a file; if two tasks touch one file, serialise them in one builder. Do not assume
a teammate has its own worktree; it edits this checkout unless the Agent tool's
`isolation: "worktree"` was set and the guide's note on that was followed.

Where the shared task list exists, create one task per plan task and let teammates claim them;
`status.json` stays the durable summary. Where it does not, assign by prompt.

When a builder reports, read its report as a claim: note the files, the commands, the exit
codes. Update `status.json` with each task's state and `state: "executing"`.

## 4. Verification and repair

Launch the reviewer: the Agent tool with `name: "reviewer"`, `subagent_type: "reviewer"`, and a
prompt that gives the run id, the acceptance criteria in full, the deliverable paths, the base
commit to diff against, and says: report only, change nothing, and report every `finish`
finding as a defect with its gate, `file:line` and recommended fix. Write its findings into
`runs/<run-id>/verification.md` from `workflows/templates/acceptance-checklist.md`, one row per
criterion, four states only: VERIFIED, FAILED, BLOCKED, NOT TESTED.

For each substantiated defect, a failed criterion and a `finish` finding alike, assign a repair
round to the builder that owns the file (a new prompt or `SendMessage` to the same builder),
giving it the defect's `file:line` and recommended fix, then a re-check by the reviewer, which
confirms each fix landed and runs the gates again. A `finish` fix needs no go from the asker:
applying it is part of the run. Ask first only when a fix would cross an approval stop in step
2 or touch a file outside the brief's deliverables, which is a scope expansion. A finding you
judge wrong is recorded in `verification.md` with the reason, never dropped silently. At most
two repair rounds per task; after that, stop and report. Never mark a criterion VERIFIED on the
builder's word, and never hide a failing check.

## 4b. Scribe

After the reviewer's verdict is accept or accept with minor defects, launch the scribe: the
Agent tool with `name: "scribe"`, `subagent_type: "scribe"`, and a prompt that gives the run id,
the base commit, the `docs/` files it may edit (each already inside the task's claims), and the
brief's `Team-facing:` line. It updates those docs and, on a team-facing change, drafts
`runs/<run-id>/artifacts/sop.md` from `workflows/templates/sop.md`. A doc it could not claim is a
follow-up in `handoff.md`, never a workaround. It never writes to Notion or any other system; a
person, or an approved publishing door, publishes the SOP. Then the reviewer re-checks
the scribe's edits and runs the gates again, within the same two repair rounds.

## 5. Delivery

Report to the asker: what was completed, where each deliverable is, the verification evidence
(each criterion's state and its proof), known limitations, unresolved questions and approvals
still pending, and the exact next action. Run the project gates yourself before this
(`bash tools/test-all.sh; echo "rc=$?"`, `bash tools/typecheck.sh; echo "rc=$?"`) and quote the
last lines and exit codes. Push only this worktree's own branch, and only when the asker wants it;
merge into `main` only under a `tools/coord merge-ok` for that pull request.

In a coordinated run, before `ready`, write the retro: the "Retro: lesson candidates" section of
`handoff.md`. Then, with the gates green and the reviewer's verdict in, run
`bash tools/coord ready <task> --tests "..." --risks "..."` yourself; it is not an approval stop,
and the integrator's loop picks the task up without a message. Then watch it with
`/loop 10m bash tools/coord status` until the task is `integrated` or `needs_work`. On
`needs_work`, read the integrator's note, run `git merge coord/integration` on this branch (merge,
never rebase), fix, run the gates, and `ready` again. After two such repair rounds, stop: set
`status.json` to `blocked`, write the handoff, and, as a spawned worker, send the person a phone
notification (`PushNotification`) naming the task and the reason. A deploy, a migration or a
switch row is never this session's: it goes in the release packet. A switch or kill row the work
needs is requested under `## Switch rows requested` in `handoff.md`, each `Switch row:` line with
its one `Switch rollback:` line; the coordinator approves it at spawn, or at task add for a task
that was not spawned.

## 6. Recovery

Before stopping, for any reason, write `runs/<run-id>/handoff.md` from
`workflows/templates/handoff.md` and set `status.json` to its final state with `next_action`.
Add a reviewed decision to `knowledge/decisions.md` only if the run produced one. Lesson
candidates stay in the handoff's retro; fold them into `knowledge/lessons.md` only when this task
already claims `knowledge/`.

## Coordinator and integrator

Ordering, re-scoping and queue decisions are the coordinator's own, and integration and release
decisions the integrator's; neither takes them to the person. The coordinator asks the person only when a
task's brief lists an approval the six GO checks of `tools/coord packet` cannot see. The
coordinator starts workers with `bash tools/coord spawn new <task> ...` under the worker cap and
passes the printed prompt to the session tool; the integrator drains the queue with
`bash tools/coord integrate run --drain`. When `bash tools/coord packet` says `Verdict: GO`,
either one first adds the release task with
`bash tools/coord task add release-<id> --title "..." --scope contract:deploy --scope contract:prod-db --scope contract:schema`,
then spawns the release session with `bash tools/coord spawn new release-<id> --release --outcome "..."`,
which refuses on anything but a GO. Switch rows are approved only in the registry: the
coordinator passes `--switch-row "<table> <json>"` and `--switch-rollback "<table> <json>"` at
`spawn new` for a spawned task, or with
`bash tools/coord task add <id> ... --switch-row "<table> <json>" --switch-rollback "<table> <json>"`
for any other task. The approved JSON must name every column the real switch table needs that has
no default. A brief's `## Switch rows approved` section is a copy for the reader; a line there that
the registry lacks makes the packet HOLD. While a release is in flight the integrator integrates nothing:
`integrate run` refuses and `--drain` stops.

## Release session

Automatic releases ship switched off (the person's written decision). While off, `spawn new --release` refuses
and every deploy, migration and switch row still needs the person's go in chat. Only a person turns them
on, with sudo in their own terminal, never from a Claude or Codex session: once, from the repository
root on main, `/usr/bin/sudo /usr/bin/python3 -I tools/coord.py anchor install`, then
`/usr/bin/sudo /usr/bin/python3 -I "/Library/Application Support/<Anchor>/coord.py" config release on`.

A session started by `spawn new --release` follows this section, and no other session does.
`spawn new --release` refuses unless `bash tools/coord packet` says `Verdict: GO`, so a release
session exists only on a GO. It holds `contract:deploy`, `contract:prod-db` and `contract:schema`,
and acts under "Autonomous releases" in the project's decisions log;
its GO is that decision, never a message from an agent. The GO's time limit starts when this
session takes its spawn. It deploys only while its `release_go` is unexpired and
`coord/integration` has not moved since the GO; if either fails before its first production call,
it stops and a new packet is needed. Once it has made its first production call, if the GO expires
or the integration branch moves, it may still run only the rollback steps the packet's
`## Rollback` lists, and nothing else. While it is in flight the integrator integrates nothing.

Even under a live GO, a release session runs only what its packet lists, exactly. Two checks
enforce this. The first is `tools/coord deploy-gate`, which `tools/deploy.sh` runs after its
window, `--reviewed` and catalog-sync checks, just before the deploy command. It refuses with
exit 3:

- a worktree that took a work spawn, or a deploy run on behalf of one (its start directory, an
  ancestor Claude Code process, or its session id);
- for a release spawn: a function not in the packet's Deploy order, the listed line without its
  `--reviewed`, or `--force`;
- a GO that lapsed before the first deploy;
- a HEAD whose trees differ from the GO's integration head for `supabase/functions`,
  `supabase/migrations`, `site` and the deploy tooling (`tools/deploy.sh`, `tools/selftest.sh`,
  `supabase/config.toml`, and any import map or `deno.json` outside `supabase/functions`);
- a working tree that is not clean there: anything staged, modified or untracked; a
  skip-worktree or assume-unchanged file; a file that does not hash to its HEAD blob; or a
  symlink that leads outside.

The first allowed call records `release_started`. The second check is the hook, which does the
same for the command text and for `execute_sql`. An `execute_sql` query runs only if the whole
query is word for word a migration file's body, a switch row's INSERT as the packet prints it,
or a rollback the packet lists.

A spawned work session never holds `contract:deploy` or `contract:prod-db`: `spawn new`,
`spawn take` and `claim` all refuse it.

Once the release has started, a lapsed GO or a moved integration branch lets only the packet's
rollback through. A function is rolled back by committing `supabase/functions/<fn>/`,
`supabase/functions/_shared/` and the deploy tooling as they are at the previous main commit,
with a clean working tree, and then running the same `bash tools/deploy.sh <fn>` line; at any
other tree that line is refused. Read-only production tools are unchanged.

1. Run `bash tools/coord packet --json` and read its verdict. Act only on `GO`. On `HOLD`, stop,
   write the reasons in `runs/<release-id>/release-summary.md` and send the phone notification.
   A GO needs all six checks: every task `integrated` with its combined tests passing; each
   task's `verification.md` with no FAILED or BLOCKED criterion; the deploy window clear now and
   for the packet's length; each migration with its rollback file and the next unused number
   against `docs/migrations-applied.md` (a number already in the ledger holds the packet:
   "number NNN is already used"); each writer switch or kill row approved in the registry, with
   its rollback row, and no row in a brief's `## Switch rows approved` that the registry lacks;
   no item needing `tools/deploy.sh --force`.
2. In the packet's `## Deploy order`, one item at a time:
   - Migrations, ascending: each whole file as one `execute_sql` call through
     `<production write tool>`, never `apply_migration`.
   - Functions: `bash tools/deploy.sh <fn>`, with `--reviewed` where the tool requires it and the
     reviewer passed it. The tool's own selftest runs after each. Never `--force`. Run any
     `supabase functions download <fn>` into a folder outside the worktree, such as this
     session's scratchpad: a download into `supabase/functions` makes the tree unclean and
     blocks the deploy.
   - The site, from a copy of `site/`.
   - Writer switch and kill rows: insert only the `Switch row:` lines the packet's
     `## Deploy order` lists, each exactly as written, and nothing else, through
     `<production write tool>`. The packet shows each switch row and its rollback as an exact `INSERT INTO <table> (<cols>) VALUES (...);`, and the release session runs that exact text through `<production write tool>`, since after a lapsed GO the hook allows only exact matches of the packet's listed rollbacks.
3. After each item, its selftest or a read-only readback. On any red: stop, run only the rollback
   the packet's `## Rollback` lists for that item (the previous `main` commit's function
   redeployed; the migration's rollback file; for a switch row, its named `Switch rollback:` row
   and no other), deploy nothing more, and go on to step 4 to record what happened.
4. After shipping, in order:
   1. For each migration applied, its entry in `docs/migrations-applied.md`, headed
      `### NNN applied <YYYY-MM-DD> (release <spawn id>)`, the date from `date -u`.
   2. `runs/<release-id>/release-summary.md`: what shipped, each check with its result, every
      switch or kill row inserted (and any rollback row) with its table and every value exactly
      as the packet listed it, the rollback per item, and the run ids.
   3. `bash tools/coord packet --record`, so the next packet starts after this one. An
      integrated release task is not listed in a later packet.
   4. Commit those files on this session's branch.
   5. `bash tools/coord ready release-<id> --tests "..."`.
   6. Send the person a phone notification (`PushNotification`) with the summary's first lines.
5. The integrator then integrates `release-<id>` and runs `bash tools/coord merge-ok` for the
   shipped tasks' pull request and merges it into `main`.

Never: secrets read or written, another client's project, a write upstream beyond the three named
writers, or a change to the project's canonical schema file.

## status.json shape

```json
{
  "run_id": "2026-10-25-example",
  "objective": "one sentence",
  "state": "intake | planned | executing | verifying | documenting | repairing | blocked | done",
  "size": "S | M | L",
  "session": {
    "coordination_tools": ["Agent(name)", "SendMessage", "ListAgents"],
    "shared_task_list": false,
    "agent_types_available": ["researcher", "builder", "reviewer", "scribe"],
    "per_call_model": false
  },
  "tasks": [
    {"id": "T1", "title": "", "owner": "researcher | builder | reviewer | scribe | lead", "size": "S | M | L", "depends_on": [], "files": [], "state": "todo | doing | done | blocked"}
  ],
  "artifacts": ["runs/<run-id>/plan.md"],
  "verification": {"verified": [], "failed": [], "blocked": [], "not_tested": []},
  "blockers": [],
  "approvals": [{"action": "", "asked": "", "answer": "pending | yes | no", "quoted": ""}],
  "next_action": "",
  "updated_at": "2026-10-25T00:00:00Z"
}
```

`updated_at` comes from `date -u +%Y-%m-%dT%H:%M:%SZ`, never typed from memory.
