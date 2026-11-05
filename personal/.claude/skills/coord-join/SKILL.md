---
name: coord-join
description: Join this already-running session to the coordinated run of parallel sessions in a repository that runs the coordination tool. User-invoked only.
disable-model-invocation: true
argument-hint: "[--role worker|coordinator|integrator|observer] [--team NAME]"
---

Join this session to the coordinated run of parallel Claude sessions (docs/coordination.md in a repository that runs the coordination tool).

1. Run exactly this from this session's working directory, and nothing else first:

   python3 "$(git rev-parse --path-format=absolute --git-common-dir)/coord/coord.py" join $ARGUMENTS

   If it fails because the file does not exist or the registry is not installed, tell the user this repository has no coordinated run set up, and stop.

2. Read what it printed and follow it from now on: your role, your team, your task and claims if any, the decisions, and the coordination rules. The rules also bind every teammate and subagent this session starts.

3. If you hold no task, do not edit tracked files. If it listed changed paths and a `task add` line, show the user that list and that line, so the coordinator (or the user, in a terminal) can assign the work to you; once they have, run the `claim` command it printed.

4. Reply to the user in two or three plain sentences: what this session joined as, and what it is waiting for, if anything.
