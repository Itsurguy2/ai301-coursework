# Evidence guide: where evidence lives in a plan package

## Diagnosis and grounding

**Where it lives.** The plan's stated cause is the first paragraph of
`## Candidate plan` (usually labeled "Cause" or "Diagnosis"). The facts
it has to explain are in `## Repro evidence`: the numbered steps, the
"Actual" line, and any "Control runs". The `## Issue` section gives the
reporter's version of the symptom. Live: the cause is in plan.md, and
the repro evidence is the student's posted repro comment on the issue.

**What good looks like.** The cause names a specific mechanism (a
function, file, or condition), and every repro fact fits it. If the
repro has a control run (same steps with one thing changed), the cause
explains why the control behaves differently. A bad one blames
something a control run already ruled out, restates the symptom as the
cause ("the color doesn't update"), or cites no repro fact at all.

## Scope

**Where it lives.** The `## Candidate plan` section, in the lines
that say what is changing ("Change", "Scope", "In") and what is not
("Out", "Not in scope"). The files or areas named there are the scope.
Compare them with the cause stated in the same plan.

**What good looks like.** One bounded change: each file or area named
is something the stated cause requires, and there is an explicit
line of what stays untouched. A drive-by rewrite shows up as work the
cause does not need: a refactor, a migration, a new abstraction,
regenerated artifacts, or a CI change sitting next to a small fix.

## Executability

**Where it lives.** The "Change" or "Approach" part of
`## Candidate plan`, read with `## Repo facts` to see if the named
places make sense in this repo.

**What good looks like.** A stranger could open the repo and start
after reading it: it names where to change (file, function, setting)
and what the change is. Numbered steps that each name a place count.
"Investigate and fix", "improve handling", or a plan that leaves a
design choice open does not.

## Test plan

**Where it lives.** The "Test" or "Test plan" part of
`## Candidate plan`. Match it against the numbered steps and the
"Expected" and "Actual" lines in `## Repro evidence`.

**What good looks like.** It re-runs the repro (or an automated test
built from it) and names a result you can see that differs before and
after the fix: output, exit code, visible behavior. A manual repro
re-run is fine. "Verify it works" or "add tests" with no named result
is not, and neither is a test that would pass without the fix (for
example, a docs-only plan tested by "the docs build").

## Honesty

**Where it lives.** Any "Risks", "Unknowns", or "Not yet checked"
lines in `## Candidate plan`, and claims in both the plan and
`## Candidate plan comment` about behavior that `## Repro evidence`
did not test. Live: a `## Deviations` section at the end of plan.md
after the build.

**What good looks like.** Anything the repro did not establish is
called an unknown or a thing to check. False confidence is a flat
claim ("this will fix all platforms") about something never tested.
A deviation from the plan is written down with what changed and why,
not left only in the diff.

## Comms

**Where it lives.** `## Candidate plan comment`, read against
`## Thread highlights` (what maintainers asked, claimed, or settled)
and `## Repo facts` (bug template, contribution policy, any AI-use
disclosure rule). Live: the issue thread on GitHub, and the repo's
CONTRIBUTING.md and issue template.

**What good looks like.** The comment answers any explicit maintainer
direction in the thread, for example a requested approach, a named
culprit, or a request for testing, and follows every repo rule that
applies, including AI disclosure if the policy requires it. A comment
that proposes a different approach than the maintainer pointed to
without addressing it fails. If the thread has no direction and the
repo states no rule that applies, a plain comment is fine.
