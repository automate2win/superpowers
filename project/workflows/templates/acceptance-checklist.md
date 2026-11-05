# Acceptance checklist: <run-id>

Filled by the lead from the reviewer's report. The reviewer checks; the lead records. Every row
cites evidence a third person can re-run or open. Four states only.

| # | Criterion (from brief.md) | State | Evidence | Checked by | When (UTC) |
| --- | --- | --- | --- | --- | --- |
| AC1 | <criterion> | VERIFIED / FAILED / BLOCKED / NOT TESTED | <command, exit code, last line, or file:line> | reviewer | <date -u> |

State meanings:

- VERIFIED: the reviewer ran the check and saw it pass. Evidence quoted.
- FAILED: the reviewer ran the check and saw it fail. Evidence quoted; a defect row below.
- BLOCKED: the check could not run for a reason outside the run (missing access, approval not given, environment). Reason stated.
- NOT TESTED: the check could have run and did not. Reason stated, and the exact manual step to run it.

## Defects

| # | Severity | Criterion | Observed | Reproduce | Owner | Round |
| --- | --- | --- | --- | --- | --- | --- |
| D1 | blocker / major / minor | AC<n>, or finish:de-slop / finish:thermo / finish:simplify | <what was seen, with file:line and the recommended fix for a finish finding> | <steps> | builder | 1 of 2 |

Every `finish` finding the reviewer reports is a defect and goes to a builder. A finding the lead
judges wrong is listed here with the reason instead of a repair:

| # | Finding | Gate | Why not applied |
| --- | --- | --- | --- |

## Gates

| Gate | Command as run | Exit | Last line |
| --- | --- | --- | --- |
| tests | `bash tools/test-all.sh; echo "rc=$?"` | | |
| typecheck | `bash tools/typecheck.sh; echo "rc=$?"` | | |

A gate not run is listed as not run, with the reason. A typecheck exit 2 is COULD NOT CHECK.

## Verdict

accept / accept with minor defects / reject, in one sentence, with the repair round count used.
