---
name: finish
description: Run the full post-implementation gate chain over a branch — QA, de-slop, thermo-nuclear review, code simplifier — as subagents, applying the recommended fixes at each gate instead of asking. Use after /execute-plan, or after any substantial change, when the work is written but not yet trustworthy.
user-invocable: true
---

# Finish

Four quality gates over the current branch. Every gate's *review* runs in a
subagent; the main session applies the fixes. This is the chain that otherwise
gets typed out one prompt at a time.

## Project facts this skill needs

This skill is shared across projects and holds no commands, model names or
branch names. Establish them, in this order: a project-local skill of the same
name (it overrides this file entirely); the project's `CLAUDE.md` /
`CLAUDE.local.md` / `AGENTS.md`; `package.json` scripts (or the stack's
equivalent: Makefile, pyproject, Cargo, CI workflow); and only then the user,
once. Record anything durable in a project file, never here.

- **Base branch** — usually `main`; check `git symbolic-ref refs/remotes/origin/HEAD`.
- **Quality floor** — the project's format, lint, typecheck, dead-code and
  schema checks, and its test command. Prefer the context-efficient wrapper
  if the project has one (e.g. a `run_silent` helper).
- **Model routing** — which model the project wants for reviews and for a
  second opinion. Defaults below apply when the project says nothing.
- **Whether QA applies** — a dev server and browser-checkable QA items. If
  not, gate 1 is skipped with a reason.

## Usage

/finish docs/my_feature_PLAN.md            # full chain
/finish                                    # find the plan in history; gates that need it degrade to the diff
/finish docs/plan.md --from thermo         # resume at a gate
/finish docs/plan.md --skip qa             # no dev server / no QA items
/finish docs/plan.md --codex               # add a Codex CLI second opinion to the thermo gate
/finish docs/plan.md --dry-run             # report findings at every gate, apply nothing
/finish docs/plan.md --serial              # no subagents; run every gate in this session

## Who does the work

**Reviews go to subagents. Fixes land in the main session.** Two reasons, and
they are the whole design: parallel agents editing one checkout collide, and
whoever applies a finding should have read the code it points at rather than a
summary of it.

The exception is QA — it has to actually drive a browser, and the only thing it
writes is checkboxes in the plan file, so it runs end to end in its agent.

Models follow the project's model-routing rules when it has them. Otherwise:
reviews are judgment work, so use the strongest Opus model available; a cheaper
model here means shipping findings that are wrong.

| # | Gate | Skill | Runs as | Model |
|---|------|-------|---------|-------|
| 1 | QA | `qa` | subagent, executes | sonnet |
| 2 | De-slop | `de-slop` | subagent, review only | opus |
| 3 | Thermo-nuclear | `thermo-nuclear-code-quality-review` | subagent, review only | opus |
| 4 | Simplify | `code-simplifier` | subagent, review only | opus |

Every review subagent gets the same standing instruction: **report findings,
change nothing.** State it in the prompt — these skills apply fixes by default
and will happily edit the tree out from under you.

With `--codex`, gate 3 also spawns a Codex CLI second opinion, using the
model the project names (else Codex's default): a thin sonnet wrapper that
writes a self-contained prompt to `$PROMPT_FILE` and runs
`codex exec -C "$REPO_ROOT" [-m <model>] review --base <base>`, read-only, with
a timeout well past ten minutes. If `codex` is not installed, say so and skip
the second opinion. Merge its findings with the Opus agent's;
where they disagree, say so rather than picking silently.

## Ordering

Gates 1 and 2 are serial and come first. QA proves the feature works before
anyone restructures it, and de-slop *deletes* — running it first means the
structural gates aren't reasoning about code that's about to disappear.

Gates 3 and 4 both review the de-slopped tree read-only, so **fan them out in
parallel** — one message, both Agent calls. Merge their findings, dedupe the
overlap (thermo and the simplifier will both flag the same nested ternary), then
apply in one pass. Commit them separately anyway, so the ledger stays honest.

## Delegated authority

**Each of these skills has a step where it lists findings and asks which to fix.
You have already been told: fix them. Do not ask.** Apply every finding you
judge correct, and say in the gate report which you declined and why.

Two things still stop you and hand back:

- A finding that changes intended behavior, a public contract, a migration, or
  anything the plan's Open Questions or the PRD owns. Report it, don't do it.
- Any of `/execute-plan`'s catastrophic failures (missing service, credentials,
  architectural ambiguity, second failure after a retry).

`--dry-run` revokes the authority: report at every gate, apply nothing.

## Before the first gate

- Resolve the plan file (argument, then conversation history). No plan is not
  fatal: gate 1 is skipped and gates 2–4 run against the diff.
- Confirm every implementation task in the plan is `- [x]`. If unchecked tasks
  remain, stop — `/execute-plan` isn't done, and the gates will review
  half-written code.
- `git diff --stat <base>...HEAD`. State the scope you are about to review.
- Run the quality floor once for a baseline. A check already red on arrival is
  not yours to chase — record it, so a later red is attributable to a gate.

## At each gate

1. Spawn the review subagent (or run the skill here under `--serial`). Give it
   the skill name to invoke, the diff scope, and the report-only instruction.
   Do not paraphrase the skill's own rules into the prompt — let it load them.
2. Take its findings. Apply them under the authority above.
3. Run the quality floor. Anything newly red is this gate's own damage — fix it
   now, never leave it for the last gate.
4. Commit: `chore(<gate>): <one line>`. Stage files individually, never
   `git add .`.
5. Append the ledger line. Report findings raised / applied / declined, and the sha.

## After the last gate

Gates 2–4 edited code that gate 1 already signed off on. So:

- Re-run `/qa` — in a subagent again — for every QA item whose files the gates
  touched. A previously passing item that now fails gets its `- [x]` cleared and
  a `> FAIL:` line, exactly as `/qa` writes it, and you stop and report. Do not
  start fixing.
- Run the full quality floor one last time.
- Output the summary table, then a copy-pastable `/pre-pr`.

## Quality floor

The project's own checks, found as described under *Project facts*. Never
`--no-verify`. If the project defines no floor, use what its CI runs; if there
is no CI either, run whatever format, lint, typecheck and test commands its
manifest exposes and say which you chose.

Add the project's test command after any gate that changed logic rather than
shape.

## Ledger

So a killed run resumes instead of restarting. Append under a
`## Finish Gates` heading in the plan file, creating it if absent:

```markdown
## Finish Gates

- [x] QA — 7 items, 7 pass — 2026-09-21
- [x] De-slop — 9 raised / 8 applied / 1 declined (real variation point) — a1b2c3d
- [ ] Thermo-nuclear
- [ ] Simplify
```

On `--from <gate>`, or when the ledger already exists, start at the first
unchecked gate and say so: "Resuming at Thermo-nuclear (2/4 gates done)."

## Rules

- One fix pass per gate. Thermo and the simplifier will find more work on a
  second look at their own output; that is a new `/finish` run, if ever.
- Gates never rewrite the plan's Implementation Plan tasks. They touch code, the
  ledger, and QA checkboxes — nothing else in the plan.
- Never skip a gate silently. A skipped gate is an unchecked ledger line with a
  reason.
- Never use Haiku for a gate. Never use Fable for a subagent unless the user asks.
- State assumptions rather than asking. The user interrupts if they disagree.
