# Evidence guide: where evidence lives in a PR package

## Plan fidelity (harness category: silent-drift)

**Where it lives.** Eval: the plan's scope is in `## Plan context`,
in the lines labeled "Scope", "Files", and "Not in scope" (and any
"Deferred", "Known limit", or "Deviation" note). What actually
changed is in `### Diff` under `## Candidate PR`: read the `+++`
file headers and every hunk. The description's claims about the
change are in `### Title` and `### Description`. Live: the scope is
the `## Scope` and `## Files I will touch` sections of `plan.md`,
and deviations are its `## Deviations` section; the diff is
`git diff main...HEAD`; the description is the draft PR text.

**What good looks like.** Every hunk is the planned change, its
test, or a file the repo requires, and every promised change is in
the diff or written down as deferred in the plan and the
description. Drift in the "more" direction is a hunk the plan never
named: a new config option or flag, a rename pass, a rewrite of a
neighboring function, or a file the plan never mentions. Drift in
the "less" direction is a promised change (a docs update, a second
fix) missing with no deferral note. Either one under a description
that claims "exactly as planned" or "no other changes" is the clearest
case. A missing piece the plan defers and the description restates
is an honest shortfall, not drift.

## Test evidence (harness category: not-tested)

**Where it lives.** Eval: `### Test evidence` under `## Candidate PR`,
plus any before/after blocks in `### Description`. Hold it against
the "Test plan" sentence at the end of `## Plan context` and the
"Repro evidence" paragraph, which names the failing input and any
controls. Live: the Testing section of the draft description, held
against the `## Test plan` section of `plan.md` and the repo's
`make` targets or test commands.

**What good looks like.** The same input that failed in the repro is
run again, before and after, with the output shown, and the after
matches what the plan's test plan expected. Every failure mode the
test plan names gets its own re-run. The repo's own suite or check
command is named with its result. Not decisive: "tested locally",
"works on my machine", "colors work now" with no output; a re-run of
only the control (the path that never broke) or of a different
input; a new test that covers something other than the fix; one of
two named failure modes shown and the other silent.

## Diff quality (harness category: unreviewable)

**Where it lives.** Eval: the `### Diff` block (added lines start
with `+`) and the `### Commits` list. Live: `git diff main...HEAD`
and `git log --oneline main..HEAD`.

**What good looks like.** Every added line is the fix, its test, or
a repo-required file, so a reviewer sees the change and nothing else.
Debris tells: `print`/`eprintln!`/`console.log`/`dbg!` or debug log
lines; lines that start with a comment marker but contain code
(`// let x = ...`, `# old_call()`); a new function nothing calls, a
`_unused` name, `#[allow(dead_code)]`; TODO/FIXME left on unrelated
lines; hunks that only re-indent, re-order or regroup imports, or
remove and re-add identical lines. Commit messages like "wip",
"fix", or "misc cleanups while debugging" point at debris but are
not debris on their own. A comment explaining why the fix works is
not debris.

## Standards and comms (harness category: standards-wall)

**Where it lives.** Eval: `## Repo facts`, in the "pull requests:"
line (the template's sections and checklist, issue-link format,
changelog or whatsnew ask, title convention, test command) and the
"contribution policy:" line (any AI policy and what it requires).
Maintainer direction is in `## Thread highlights`. Where the PR
honors them: `### Description` for sections, checklist answers,
issue links, and AI disclosure; `### Diff` for a changelog or
whatsnew file. Live: `.github/PULL_REQUEST_TEMPLATE.md`,
`CONTRIBUTING.md`, the house rules in `scope.md`, and the issue
thread, held against the draft description and the diff.

**What good looks like.** Each concrete ask that applies has visible
content: the named sections are there with real text, the checklist
items are answered, "closes #N" or "fixes #N" is present when asked,
the changelog/whatsnew file is in the diff when asked. When the repo
states an AI policy, the description carries a disclosure that says
what the policy asks for (tool and extent, or own-words review). A
visibly ignored ask is a description with no issue link, no
checklist, and no whatsnew entry when the template requires all
three, or no AI disclosure when the policy requires one. Asks that
do not apply (screenshots on a non-UI fix) are not missing. Whether
the description's claims match the diff is plan fidelity, above.
