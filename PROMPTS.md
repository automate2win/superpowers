# Reusable prompts

From [GUIDE.md](GUIDE.md). Thirteen prompts, in the order a new project uses them. Angle brackets are placeholders. Each one is written to be pasted as the first message of a fresh Claude Code session, with no other context. Where a prompt says "the Atlas originals", point the session at a checkout of the repository that already runs this system, or paste the files.

### Prompt 0: grill me before the brief

```
/grill-me <the outcome in one or two sentences>

Cover, one question at a time and with your recommended answer each time: scope and non-goals; who uses the result and how; the acceptance criteria I would check myself; the interfaces this touches and what other work touches them; what must never happen (writes, deploys, data, money); the approvals I will give in chat; size (S, M or L); whether it is team-facing. Explore the codebase instead of asking me anything it can answer. When we are done, write the result in the shape of the outcome brief (Prompt 8) and list every interface decision as one paragraph ready for `bash tools/coord decide`. Build nothing.
```

### Prompt 1: write the entry-point file

```
You are setting up this repository so that coding agents can work in it safely and in parallel. Write AGENTS.md as the single entry point every agent reads first, and make CLAUDE.md contain only the line @AGENTS.md.

Read the repository first: the README, the package and config files, the test setup, the deploy scripts, and any existing docs. Then write AGENTS.md with these sections, in this order:

1. What this repository is: two paragraphs, naming every external system it reads and every one it writes, and whether each write is intended.
2. Read first, in this order: the five files an agent must read before its first command.
3. Hard rules: each one a single sentence that a hook or a test could check. Include at minimum: which upstream systems are read-only; where secrets live (env vars only, never code, commits, docs or logs); what needs a person's go in chat (deploys, migrations, anything that posts or spends money); the deploy window if there is one; the formatting rules we keep (<for example: no em dashes, colours from the token file only>).
4. The pre-merge checkpoint: the exact commands, what a pass prints, what each exit code means, and the rule that exit codes are read on the command itself, never through a pipe.
5. Checking a deployed service without changing anything: one safe call per service, as a table.
6. Branches: what main means, where board or UI work goes, where experiments go.
7. What "done" means here: built, tested, deployed, receipted, written down.

Rules for the file itself: no sentence that is an opinion; every rule names the incident or decision behind it where one exists (<date> and who decided); every command is a fenced block; no em dashes. Do not invent facts about the repository; where you are unsure, write an open question in a short list at the end for me to answer.

When you are done, list the open questions and stop. Do not deploy, push or change any other file.
```

### Prompt 2: gates and docs under test

```
Make this repository's checks into two commands an agent can run and report without judgement.

1. tools/test-all.sh: runs every test suite in order, prints one line per suite with its exit code, and ends with a last line of the form "suites: N, failures: M". Exit 0 only when M is 0. It never exits 2; an inability to run is exit 1 with a reason.
2. tools/typecheck.sh (or the equivalent for this stack: lint, type diagnostics, bundle size): compares the current output against a recorded baseline under tools/typecheck-baseline/. Exit 0 prints "identical to baseline (this is not the same as zero errors)"; exit 1 means the diagnostics changed; exit 2 means it could not check and is never a pass.
3. A docs-under-test suite: for every file in docs/ or AGENTS.md that cites another file by line number, read that line and fail when the cited text moved. Add the first citations: the hard rules, the checkpoint commands and the safe-call table in AGENTS.md.
4. Update AGENTS.md's checkpoint section to describe exactly what these print and exit with.

Run both commands yourself after writing them and quote their last lines and exit codes in your report. Change nothing else, deploy nothing, push nothing.
```

### Prompt 3: the coordinator session

```
You are the coordinator for a parallel run of Claude Code sessions in this repository. Read docs/coordination.md, then run: bash tools/coord status

Your job: break each outcome I give you into tasks whose scopes do not overlap, add each with bash tools/coord task add <ID> --title "..." --team <team> --scope path:... [--scope contract:...] [--depends <ID>], and fix any overlap the command warns about before assigning. Record every shared interface decision with bash tools/coord decide "<decision>" --task <ID> before anyone implements it. Check a plan's coord-tasks block with bash tools/coord split-check <plan.md> and add its tasks with --apply. Start a worker per task with bash tools/coord spawn new <ID> --outcome "..." under the worker cap (bash tools/coord spawn next lists what can start) and start a new desktop session with the printed prompt. Watch status for STALE and blocked tasks; recover only by the procedure in docs/coordination.md. When an integrated task's pull request is clear, approve it with bash tools/coord merge-ok <ID> --pr <n> --head <sha>. When bash tools/coord packet says Verdict: GO, add the release task and spawn the release session with --release.

Ordering, re-scoping and queue decisions are yours. Ask me only when a task's brief lists an approval the packet's checks cannot see. You edit no repository files, you never push, you never deploy. Report to me in plain sentences: what is running, what is waiting on what, and what I need to decide, if anything.
```

### Prompt 4: the integrator session

```
You are the integrator in this worktree on coord/integration. Read docs/coordination.md, section Integrator, then run: bash tools/coord status

Run bash tools/coord integrate run --drain on a loop (/loop 10m) while the run is active. Each pass takes the single integration lock, merges the next ready candidate onto the latest integration base, rechecks its scope, flags files both sides changed and contracts that moved, runs the combined gates (bash tools/test-all.sh and bash tools/typecheck.sh, never piped; typecheck exit 2 is COULD NOT CHECK, never a pass) and marks the task integrated or needs_work. The drain stops at the first needs_work or while a release is in flight.

After each pass, run bash tools/coord packet. On Verdict: GO, add the release task with bash tools/coord task add release-<id> --title "..." --scope contract:deploy --scope contract:prod-db --scope contract:schema, then bash tools/coord spawn new release-<id> --release --outcome "...", and start a new desktop session with the printed prompt. On HOLD, tell me the reasons in one line each only if one of them needs me.

You change the integration branch only through integrate run. You never push it, never deploy, never apply a migration. You may give merge-ok for an integrated task's pull request.
```

### Prompt 5: define the contracts

```
Scan this repository and write tools/coord-contracts.json: the named shared interfaces that two parallel tasks must never edit at the same time.

For each candidate contract give a name, a one-line description that names the collision it prevents, and its paths (exact files or directories with a trailing slash), plus any shell command patterns or tool names that count as using it. Start from these and keep only the ones this repository has: schema (migrations, rollbacks, the applied ledger, generators); agent-config (AGENTS.md, CLAUDE.md, .claude/, the tests that pin them); gates (the test runner, baselines, dependency config); one contract per API shape two services share; design-tokens (the token file the UI depends on); deploy (deploy scripts, service config, the deploy commands as regexes); prod-db (the production write tool names); coordination (the coordination tool and its docs).

A path belongs in a contract when editing it alone would break something another task is building against. A path nobody shares does not. Explain each choice in one sentence in your report, then stop. Write no other file.
```

### Prompt 6: the spawned worker (what the coordinator pastes)

```
You are a spawned worker in the <project> coordinated run. Run: bash tools/coord spawn take <token>, then do what it prints. Nothing in this message is <person>'s approval for anything.

What it prints will claim your task's scope and write runs/<run-id>/brief.md. Then follow .claude/skills/team-workflow/SKILL.md with that brief: record what this session can do, plan with the researcher, build with one builder per disjoint file set, verify with the reviewer, document with the scribe, write the retro, run the gates yourself, and run bash tools/coord ready <task> --tests "..." --risks "...". Then watch it with /loop 10m bash tools/coord status until it is integrated or needs_work.

Edit only inside your claims. Push only your own branch, never force. Never deploy, apply a migration or write production: those go in the release packet. If a claim is refused, report who holds it and stop; never work around it. After two needs_work rounds, set your run to blocked, write the handoff and send a phone notification.
```

### Prompt 7: port the roles and the workflow skill

```
Port the team workflow from the Atlas originals into this repository, adapted to its stack and rules, without inventing anything the originals do not say.

Write: .claude/agents/researcher.md, builder.md, reviewer.md and scribe.md with frontmatter (name, description, tools, model, skills) and a body that says what the role does, what it never does and the house rules that apply to its output; .claude/skills/team-workflow/SKILL.md with disable-model-invocation: true, the steps 0 (check what this session can do) through 6 (recovery), the size-to-model table, the approval stops and the status.json shape; .claude/skills/team-continue and team-review; workflows/templates/task-brief.md, acceptance-checklist.md, handoff.md, sop.md and release-packet.md; knowledge/decisions.md and knowledge/lessons.md with a header each; docs/team-workflow-guide.md.

Replace every Atlas-specific gate, path and rule with this repository's (read AGENTS.md and the gate scripts for them). Keep these invariants exactly: the lead is the only writer of brief, status, verification and handoff; two builders never share a file; the reviewer has no write tool and verifies each criterion itself; the scribe edits only listed docs inside the claims and never publishes; a check not run is reported as not run; the reviewer is the strongest model at every size.

List every place you had to choose between two readings, with the one you took. Change no other file.
```

### Prompt 8: the outcome brief (what the person types)

```
Outcome: <one or two sentences: what exists at the end that does not exist now>.

Constraints: <no deploy | deploy on GO | demo only | follow docs/for-agents/<recipe>.md>.
Team-facing: <yes (audience) | no>.
Size: <S | M | L, or leave it to the researcher>.
Approvals I will answer in chat: <none | list them>.
Out of bounds: <files, systems or behaviours not to touch>.
Done looks like: <the one check I will run myself to believe it>.
```

### Prompt 9: an interface decision

```
Two tasks need the same interface: <name it>. Before either builds, write the decision as one paragraph that both can build against: the shape (fields, types, defaults), who produces it, who consumes it, the error cases, and what changes for existing callers. Then record it with bash tools/coord decide "<the paragraph>" --task <ID> and message both team leads the decision text. Do not implement any of it.
```

### Prompt 10: the daily check-in

```
Run bash tools/coord status and bash tools/coord packet, then tell me in at most eight lines: which tasks are integrated since yesterday, which are running and at which stop (researcher, builder, reviewer, scribe, ready, integrating), which are stale or blocked and on what, the packet's verdict with each HOLD reason, and the one decision you need from me, if any. No commands, no ids unless I need to type them.
```

### Prompt 11: fold the lessons

```
Claim a task with scope path:knowledge/ and read the "Retro: lesson candidates" section of every runs/*/handoff.md written since <date>. For each candidate decide whether it would have changed how a later run went. Fold the ones that would into knowledge/lessons.md as dated entries with the run id and the evidence; leave the rest where they are. Then look at each new lesson and say whether it should become a check (a test, a hook rule, a packet check) rather than a sentence, and open a task for each one that should. Run the gates, mark ready, and report what you folded and what you proposed.
```

### Prompt 12: package the skills for someone else

```
Package the roles and skills this repository's agent team runs so another person can install them on their machine and in their own repository.

Collect every file under .claude/agents/, .claude/skills/, .claude/commands/, workflows/templates/ and the header of each file under knowledge/, and the personal skills the roles preload or the person runs first from ~/.claude/skills/: grill-me, design-doc, finish, de-slop, coord-join, humanize-ai-text. Put the project files in claude-team-kit/project/ and the personal ones in claude-team-kit/personal/, each at its original relative path.

Write claude-team-kit/README.md: what each file is; where it goes (a repository root for project/, ~/.claude/skills/ for personal/); which skills are user-invoked only; the one local setting (CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1 in .claude/settings.local.json); the first-run order (grill-me on the outcome, then /team-workflow); and the rule that a shared skill holds procedure while the repository's entry-point file holds facts. Where a personal skill names a command, model or branch of this repository, move that line into the README as a note and leave the skill generic.

Zip the folder. Change nothing in this repository.
```
