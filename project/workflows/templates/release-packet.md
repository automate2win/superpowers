# Release packet

Rendered by `bash tools/coord packet` from integrated work not yet in `main`. An integrated
release task is not listed in a later packet. It is read-only: rendering it deploys nothing. A release session spawned by the coordinator or integrator executes
it only on `Verdict: GO`, under "Autonomous releases" in the project's decisions log.

## Go asked for

Verdict: GO | HOLD

- One line per check: every task integrated with its combined tests passing; the reviewer's
  verification with no FAILED or BLOCKED criterion; the deploy window clear now and for the
  packet's length; each migration with its rollback file and the next unused number (a number
  already in the ledger is HOLD); each writer switch row approved in the registry, with its
  rollback row, and no row in a brief that the registry lacks; no item needing `--force`.
- On HOLD, each reason.

## What changed

- `<task id>`: `<title>`, branch `<branch>`, integrated at `<short sha>`, run `runs/<run-id>/`.

## Gate receipts

| Task | Gate | Exit | Last line |
| --- | --- | --- | --- |

## Deploy order

1. Migrations, ascending, each as one `execute_sql` through `<production write tool>`, then its entry
   in `docs/migrations-applied.md`.
2. Functions, one at a time, with `bash tools/deploy.sh <fn>` (and `--reviewed` where the tool
   requires it), each followed by its selftest, from a clean working tree. Any
   `supabase functions download <fn>` goes to a folder outside the worktree; a download into
   `supabase/functions` makes the tree unclean and blocks the deploy.
3. The site, from a copy of `site/`.
4. The writer switch rows approved in the registry, each listed as `Switch row: <table> <json>`
   with its exact `INSERT INTO <table> (<cols>) VALUES (...);`, and followed by a readback. The
   release session runs that exact text through `<production write tool>`. The release inserts no other
   row.

## Rollback

| Item | Rollback |
| --- | --- |

A switch row's rollback is its named `Switch rollback: <table> <json>` row, and only that row,
shown as its exact `INSERT INTO <table> (<cols>) VALUES (...);`. After a lapsed GO the hook allows
only exact matches of the rollbacks listed here. Even under a live GO, a release session runs only
what its packet lists, exactly. A function is rolled back by committing `supabase/functions/<fn>/`,
`supabase/functions/_shared/` and the deploy tooling as they are at the previous main commit, with a
clean working tree, and then running the same `bash tools/deploy.sh <fn>` line; at any other tree
that line is refused.

## After the go

1. Each applied migration's entry in `docs/migrations-applied.md`, headed
   `### NNN applied <YYYY-MM-DD> (release <spawn id>)`.
2. `runs/<release-id>/release-summary.md`: what shipped, each check with its result, every switch
   row inserted with its table and values, the rollback per item and the run ids.
3. `bash tools/coord packet --record`, then a commit on the release branch and
   `bash tools/coord ready release-<id>`.
4. A phone notification to the person with that summary.
5. The integrator integrates `release-<id>` and runs `bash tools/coord merge-ok` for the shipped
   tasks' pull request, then the merge into `main`.
