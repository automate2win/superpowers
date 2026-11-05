# super-powers

The roles, skills, templates and prompts for running several Claude Code sessions as one team: a coordinator that splits an outcome into tasks, workers that each plan, build, get an independent review and document themselves, one integrator, and a release step that runs only on a machine-checked GO. The person writes the outcome, answers an interview about it, sets the policy, and reads the summaries.

Read [GUIDE.md](GUIDE.md) first. It has the executive summary and the diagram, the roles, the building blocks, the life of one task, the skills each role runs, the setup order for a new repository, and the risks each rule covers. [PROMPTS.md](PROMPTS.md) holds the thirteen prompts on their own.

## What is in here

| Folder | What | Goes to |
| --- | --- | --- |
| `project/.claude/agents/` | The four teammate roles: researcher, builder, reviewer, scribe | your repository root, as `.claude/agents/` |
| `project/.claude/skills/` | `team-workflow`, `team-continue`, `team-review` (user-invoked only) | your repository root, as `.claude/skills/` |
| `project/workflows/templates/` | The brief, acceptance checklist, handoff, SOP and release packet templates | your repository root |
| `project/knowledge/` | Empty `decisions.md` and `lessons.md` with their header rules | your repository root |
| `personal/.claude/skills/` | `grill-me` (and `grilling`), `design-doc`, `finish`, `de-slop`, `humanize-ai-text`, `coord-join` | `~/.claude/skills/` on your machine |
| `docs/coordinated-run.png` | The diagram from the guide | reference |

## Install

1. Copy everything under `personal/.claude/skills/` into `~/.claude/skills/`. These work in every repository on the machine.
2. Copy everything under `project/` into the root of the repository you want to run this in, keeping the paths.
3. Write the repository's entry-point file (`AGENTS.md`, with `CLAUDE.md` containing only `@AGENTS.md`). Prompt 1 in [PROMPTS.md](PROMPTS.md) drafts it. The roles and skills read project facts from there: the test command, the deploy rules, what needs a person's go.
4. Run Prompt 7 in a fresh session to adapt the project files to your stack. The copies here name one stack's paths (an edge-functions folder, a deploy script, a coordination command line) as placeholders to replace.
5. Set `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` in the repository's `.claude/settings.local.json` under `env`, and start a fresh session so the agent definitions load.

## First run

1. `/grill-me <your outcome in one or two sentences>` (Prompt 0). Answer the interview. The result is the brief.
2. `/team-workflow <the settled outcome>`. The lead does intake, plans with the researcher, builds with the builder, verifies with the reviewer, documents with the scribe, and leaves records under `runs/<run-id>/`.
3. `/team-continue <run-id>` to pick a run up later; `/team-review <run-id>` for an independent check.

Running several workers in parallel, the integrator loop, the release packet and the person-only release switch need the coordination tool described in the guide, which is not in this repository yet. Everything above works for one session at a time without it.

## The one rule that keeps this portable

A shared skill holds procedure and no project facts. It names no command, no model, no branch. The project facts live in the repository's entry-point file, and the skill reads them from there. When you change a skill, keep that split, and the same files move to your next repository unchanged.

## Notes

- The personal skills were collected and adapted over time from community sources and the author's own work. They are shared as they are used; keep any header a file carries.
- Nothing here contains credentials, project identifiers or client data. Each file was checked before publishing. If you find something that looks like one, open an issue.
