# Unit 4: Test and Submit

Path: `beat-1-sandbox/unit-4/pull-request.md`

---

## Your identity upstream

**GitHub username**

Itsurguy2

---

## Pull request

**PR URL**

https://github.com/codepath/pathreview-ai301-fa26-s3/pull/142

**Issue**

https://github.com/codepath/pathreview-ai301-fa26-s3/issues/62

**Branch**

fix/62-redis-health-url

---

## Eval iterations

**Run history**

1. Smoke run (`--only pkg-01,pkg-13,pkg-07`, partial, not saved): 3/3 agreed. I picked these three because they test the edges I was most worried about: pkg-01 (standards-wall, a good fix that skips pandas' checklist), pkg-07 (not-tested, evidence runs only the control), and pkg-13 (clear-accept with an honestly deferred `%VAR%` case). I opened the per-check results to make sure each verdict came from the right check: pkg-01 failed only `repo-asks-met`, pkg-07 failed only `repro-rerun-before-after`, and pkg-13 passed everything, with the deferral counted as a plan note rather than missing work.
2. Full run 1 (the confirming run, saved as `eval-run.txt`): 19/20, bar PASS. Categories: clear-accept 6/7, not-tested 4/4, silent-drift 4/4, standards-wall 2/2, unreviewable 3/3. The one disagreement was pkg-05.

I did not revise the skill after the full run, so there was only one full run. 19/20 with every category matched clears the bar, and the pkg-05 miss comes from a deliberate strictness in one check (see Package analysis and Trade-offs), not a check that is reading the wrong evidence.

**Package analysis**

pkg-05 (nushell #18848, same-name keybindings collapsing on merge). Gold: accept. My skill: reject.

The PR widens the merge identity to `(name, mode, modifier, keycode)` and adds a one-time warning when a merge replaces a binding. Seven of my eight checks passed: the diff matches the plan's files, the description matches the diff, the repo's suite ran (`cargo test -p nu-protocol` 312 tests, fmt and clippy clean), there is no debris, both template sections are filled, and the description engages fdncred's uniqueness question. The check that rejected it was `repro-rerun-before-after`. The plan's test plan names two scenarios: "re-run the issue's script, expect both `atuin` rows; re-run with two same-name same-key bindings, expect one row plus a warning." The evidence shows the first one before and after as a table, but the second is only one sentence: "Same-key redefine prints the one-time warning." My skill's evidence line was: "the same-key-replace-plus-warning failure mode is only narrated ... never re-run with shown output."

My rubric decided this way because its pass condition requires shown before/after output "for every failure mode the plan's test plan names". I wrote it that way on purpose to catch calib-04, where the plan names two failure modes and the evidence re-runs only one. Gold reads pkg-05 differently, and I think the difference is real: the second scenario in pkg-05 isn't a failure mode from the repro. It is a check on the new warning behavior, and the issue's actual bug (ctrl-r reverting to `history_menu`) is fully shown before and after. My check can't tell "a failure the reporter hit" from "an extra behavior the plan wants confirmed", so it treats both as owed evidence.

**Check rationale**

`repro-rerun-before-after`, as it reads in `tools/pr-precheck/rubric.md`:

> The evidence re-runs the same steps and inputs that failed in the plan's repro, for every failure mode the plan's test plan names, and shows an observable result for both before and after (output, exit code, visible behavior), with the after matching the plan's expected-after. Fail if the evidence is a statement with no shown output ("works now", "verified on my machine", "tested locally"), if it runs a different input or path than the repro (a control, a single-file case, an unchanged code path), or if a failure mode the plan's test plan names is never re-run

Each part targets a specific way the not-tested packages slip through. "The same steps and inputs that failed" is there because pkg-07 and pkg-14 both show real output, but for the wrong input: pkg-07 runs the single-file control that never broke, and pkg-14 runs a GET with header casing instead of the single-header POST. A check that only asked for "before and after output" would pass both. The quoted phrases in the fail list ("works now", "verified on my machine") are there because pkg-04 and pkg-10 have no output at all, just a claim, and listing those phrases makes the grader treat them as no evidence. "For every failure mode the plan's test plan names" is for calib-04, where the plan names two failure modes and the evidence covers one. Reading against the plan's test plan, not the PR's own idea of what to test, keeps the check about whether the PR kept its plan's promise.

**Trade-offs**

The "every failure mode" clause is strict, and it cost me pkg-05: a sound PR whose plan named a second scenario that the evidence describes in words but doesn't show. I kept it because loosening it to "the main repro is re-run" would let through PRs like calib-04, which only prove half of what they promised. The cost is that a PR whose plan over-promises on testing gets held for not showing everything, even when the issue's own bug is fully proven. For my own PRs that is the habit I want anyway: if my plan's test plan names it, my description should show it. I also chose not to spend another full run fixing pkg-05, since a change to this check could flip calib-04 or a not-tested package, and the current run already clears the bar with every category matched.
