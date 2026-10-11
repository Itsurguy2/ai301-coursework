---
name: pr-precheck
description: Grade a PR package (a candidate pull request read against the plan it claims to implement and the issue that plan belongs to) and decide whether it is ready to submit. Use when checking your own branch, draft PR title, and description before opening the pull request, or when grading an eval package bundle.
---

# pr-precheck: rubric-driven PR grading

## The question

You answer one question about one PR package: is this pull request
ready to submit? A PR package is a candidate pull request (its title,
description, commit list, diff, and test evidence), read against the
plan it claims to implement (including any deviation or deferral
notes in that plan) and the issue the plan belongs to (the issue
text, the thread, and the repo's stated PR template and contribution
policy). Do not grade whether the issue was worth taking, whether the
plan was a good plan, or whether the code is how you would have
written it. The plan is the yardstick, not the thing under review.
Grade one package per run, and never answer from gut feel: answer by
executing `procedure.md`, which applies the checks in `rubric.md`
using the map in `references/evidence-guide.md`.

## Inputs and modes

Run in exactly one of two modes.

**Live mode** is the student's own PR, checked before it is opened.
Read these inputs and nothing else as the package:

- The plan: `plan.md` in the working copy, including its
  `## Deviations` section if it has one. A house-chain student reads
  the house plan and the house repro pack instead.
- The diff: the output of `git diff main...HEAD` (three dots), run
  from the working copy on the PR branch. This is everything the
  branch changes relative to the default branch. Do not use the
  working tree or uncommitted files; untracked drafts sitting in the
  folder are not part of the PR.
- The commit list: `git log --oneline main..HEAD`.
- The draft PR title and description: the text the student gives you,
  or the draft file they name.
- The test evidence: the before/after output and check results the
  draft description quotes. Evidence that lives only in the student's
  terminal and is not in the description does not count, because the
  reviewer will not see it.
- The issue: the issue the plan names. Read its body and thread on
  GitHub (`gh issue view <n> --comments`), the repo's PR template
  (`.github/PULL_REQUEST_TEMPLATE.md`), and `CONTRIBUTING.md` for the
  stated asks.

**Eval mode** grades a package bundle. The bundle is the whole world:
every fact comes from the bundle text, nothing is fetched, and no
other file is read. Eval mode always grades a complete package: run
every check in the rubric and apply the full verdict rule. Treat
every eval package as AI-assisted work when a check reads an AI-use
policy.

## The scope seam (live mode only)

In live mode, read `scope.md` before anything else. If its `Repo:`
line still holds the bracketed placeholder (`<ORG>/<PATH-REVIEW-REPO>`
or any other `<...>` text), stop without grading and tell the student
to fill the `Repo:` line in `scope.md` with their section's Path
Review repo. Never guess a scope. If the PR's target repo or the
plan's issue is not in the scoped repo, refuse to grade and say why.
Apply the house rules the scope lists as stated asks of this repo:
they are read by the rubric's standards checks exactly like a PR
template (for example the branch name pattern, one PR per issue, and
the template used with every section filled). In eval mode, ignore
`scope.md` entirely.

## The voice seam (live mode only)

In live mode, after grading, read `voice-guide.md` and hold the draft
PR title and description against each of its rules. In the summary,
list each rule the draft breaks: quote the rule's name and the words
in the draft that break it. If the draft breaks none, say so in one
line. The voice guide never changes the verdict on its own, because
voice is the student's personal standard and no rubric check reads
it; a broken voice rule is advice, not a hold. In eval mode, ignore
`voice-guide.md` entirely.

## Component reads

- `rubric.md` defines the checks (name, evidence, pass condition,
  weight) and the verdict rule. It is the only source of checks: do
  not add a check it does not list, and do not skip one it does.
- `references/evidence-guide.md` maps where each evidence family
  lives in a bundle and in live mode, and what good looks like. Use it
  whenever a check's evidence column names a family.
- `procedure.md` is the operating procedure. Execute it as written:
  its read order, its gathering steps, its check execution, its
  verdict assembly. Where it is silent on a step you need, do the
  narrowest thing the rubric's text supports and report the gap in
  your summary as "Procedure gap: ...". Never invent a new step and
  present it as the procedure.

If `rubric.md` has no checks or no verdict rule, or `procedure.md`
has no steps, stop and say which file is empty. This tool does not
grade without a rubric and a procedure, and that is by design.

## Verdict and output

The verdict is binary. `accept` means the PR is ready to submit;
`reject` means hold it and fix what the failing checks name. There is
no third verdict and no score. Reservations go in a check's evidence
line, not in the verdict.

Before the JSON block, write a short readable summary: one line per
check with its grade and the deciding fact, then the verdict and the
deciding check, then (live mode) the voice-guide notes and any
procedure gaps. End the reply with this fenced JSON block, valid, and
put nothing after it:

```json
{
  "item": "<PR URL or bundle id>",
  "checks": [
    {"name": "<check name>", "grade": "pass|fail|unclear",
     "evidence": "<one line: the fact or quote that decided it>"}
  ],
  "verdict": "accept|reject"
}
```

Use the bundle id as `item` in eval mode and the PR URL (or the
branch name, if the PR is not open yet) in live mode. List every
rubric check, in rubric order, under its exact rubric name.

## Grading discipline

- Evidence first. Never grade a check without naming the fact or
  quote that decided it: a file path, a hunk, a sentence from the
  description, a line of test output. "Looks fine" is not evidence.
- Grade the thing, not the polish. A terse PR that does what the plan
  says, shows it, and meets the repo's asks is ready. A long,
  confident, well-formatted one can still hide drift or skip the
  repo's asks. Read the diff against the plan yourself; never take
  the description's word for what the diff contains.
- The rubric decides, not the run. If a check passes by its stated
  condition but feels wrong, it passes; note the tension in the
  summary.
- The procedure decides how, not the run. Follow `procedure.md` and
  report its gaps instead of working around them.
- Honest shortfalls are not failures. Work the plan or its
  deviation notes deliberately defer, and that the description
  restates as deferred, is not missing work.
- Unclear counts as fail unless the rubric's verdict rule says
  otherwise. A claim you cannot verify from the package is a failing
  claim: a PR you cannot verify is not ready to submit.
