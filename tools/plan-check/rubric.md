# Rubric: is this plan ready to post and build from?

## Checks

| Check | Evidence | Pass condition | Weight |
|---|---|---|---|
| diagnosis-matches-evidence | The plan's stated cause, read against the Repro evidence block (steps, control runs, observed output) and the issue text | The cause names a specific mechanism (a function, file, or condition) AND is consistent with everything the repro evidence shows, including any control run that rules a suspect in or out. Fail if the cause contradicts the repro evidence, blames something the evidence rules out, restates the symptom as the cause, or cites no repro fact at all | required |
| scope-bounded | The plan's Change/In/Out statement read against the diagnosis and the issue's reported problem | The plan names each file or area it will change, says what it will not touch, and every named change follows from the diagnosed cause. Fail if the plan adds work the diagnosis does not require (refactors, migrations, redesigns, new features, "while we're here" fixes), or bundles the bounded fix inside a larger campaign, or names no files and no exclusions | required |
| executable-by-stranger | The plan's change and approach statements, read with the repo facts | A developer who has never seen this issue could start the change from the plan alone: it says where to change what, and the change is one coherent piece of work. Fail if the approach is a goal with no location ("improve the handling", "investigate and fix") or depends on a decision the plan never makes | required |
| test-observes-outcome | The plan's test plan read against the repro steps in the Repro evidence block | The test re-runs the repro (or an automated test built from it) and names an observable result that differs before and after the fix (an output, exit code, or visible behavior). Manual or automated both pass. Fail if no observable outcome is named ("verify it works", "add tests") or the test would pass without the fix | required |
| thread-and-conventions | The plan comment read against the Thread highlights block and the Repo facts block (contribution policy, AI policy, templates) | The comment responds to any explicit maintainer direction in the thread (a requested approach, a claimed fix, a request for testing) instead of ignoring it, and honors every stated repo convention, including any AI disclosure the policy requires. If the thread has no direction and the repo states no applicable rule, pass | required |
| unknowns-stated-honestly | The plan's risks and unknowns, and any claims about untested behavior, read against the repro evidence | Anything the repro evidence does not establish is stated as unknown or as a check to perform, not asserted as fact. Fail if the plan states an untested claim as certain | preferred |

## Verdict rule

Accept if every required check passes. Reject if any required check
fails. A required check graded unclear counts as fail, because a plan
that cannot be verified from the package is not ready to build from.
The preferred check never changes the verdict. Where the diagnosis
conflicts with the repro evidence, reject regardless of every other
grade.
