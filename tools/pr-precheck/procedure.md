# Procedure: how this tool grades a PR package

## Read order

1. Read the issue. Note in one sentence what is broken and what the
   reporter expected.
2. Read the thread highlights. List each explicit maintainer
   question, requested approach, or stated rule. If there are none,
   write "no direction".
3. Read the repo facts. Copy every concrete PR ask into a list: each
   named template section, each checklist item, any issue-link line,
   any changelog or whatsnew entry, any title convention, any test
   command. Separately, copy the AI policy sentence, or write "no AI
   policy".
4. Read the plan context: first its repro evidence, then the plan.
   Write down three lists: (a) the changes and files the plan says it
   will make, (b) what it says is out of scope or deferred, and
   (c) every failure mode and expected-after its test plan names,
   plus the suite or check command it names.
5. Read the diff, hunk by hunk. For each hunk write one line: file,
   what the hunk does, and whether it matches list (a). Do this
   before reading the description.
6. Read the commit list.
7. Read the description and title last, then the test evidence.

Why this order: the plan and the diff are the facts, and the
description is a claim about them. If you read the description first
you will read the diff through its words and miss a hunk it does not
mention. Reading the plan's test plan before the evidence means you
already know which failure modes the evidence owes.

## Evidence gathering

1. diff-matches-plan: from step 5's hunk list, mark each hunk as
   "planned", "test for the fix", "repo-required file", "deviation
   note covers it", or "unplanned". Then go through plan list (a) and
   mark each promised change "in diff", "deferred in plan", or
   "missing". Record every "unplanned" and "missing" item with its
   file.
2. description-matches-diff: copy each sentence of the title and
   description that says what changed or claims fidelity ("exactly",
   "no other changes", "now documents"). Next to each, write the hunk
   that makes it true, or "not in diff", or "diff does more".
3. repro-rerun-before-after: for each failure mode in plan list (c),
   find the evidence block that re-runs it. Write the command or
   steps used, whether they are the same input as the repro or a
   different one (a control, a smaller case, a different path), the
   shown before output, and the shown after output. Write "none
   shown" for any part that is only described in words.
4. repo-checks-run: find the line in the test evidence that names a
   suite or check command and its result. Copy it, or write "none".
5. diff-free-of-debris: scan every added line in the diff for debug
   output, commented-out code, dead code, stray TODO/FIXME, and hunks
   that only re-indent, re-order imports, or re-print unchanged
   lines. Copy each one with its file. Note commit messages like
   "wip", "fix", or "misc cleanups" as a pointer to look harder, not
   as a fail on their own.
6. repo-asks-met: for each ask in the step 3 list, find where the
   description or diff meets it and quote it, or write "absent". Mark
   asks that do not apply to this PR as "n/a" with the reason.
7. ai-disclosure: if step 3 found an AI policy that asks for
   disclosure, find the sentence in the description that discloses AI
   use and compare it with what the policy asks. Otherwise write
   "no AI policy".
8. thread-direction-engaged: for each item from step 2, find where
   the description answers it, or write "not addressed".

## Check execution

1. Run the checks in rubric order, one at a time, using only the
   gathered evidence and the rubric's pass condition.
2. Grade fail when the gathered evidence shows the pass condition is
   broken, and put the breaking fact in the evidence line (the file
   and hunk, the quoted claim, the missing failure mode, the absent
   ask).
3. Grade unclear only when the package lacks the part the check
   reads (for example, no test evidence section at all, or no diff).
   Look again before grading unclear.
4. Grade each check on its own. A check that fails does not make
   another check fail, and a strong result on one check does not
   excuse another.
5. Do not give credit for length, headings, confidence, or neat
   formatting, and do not take it away for terseness.
6. A deferral counts only when it is written in the plan (or its
   deviation notes) AND restated in the description. A gap that
   appears only in the diff is not a deferral.

## Verdict assembly

1. List the grades of the seven required checks in rubric order.
2. If all seven are pass, the verdict is accept.
3. If any is fail or unclear, the verdict is reject. Unclear counts
   as fail.
4. Report the preferred check's grade but never use it in the
   verdict.
5. For a reject, the deciding check is the first failing or unclear
   required check in rubric order. Name it first in the summary and
   quote the fact that broke it. Name any other failing checks after
   it.
6. End with the JSON block, listing all eight checks in rubric order,
   and nothing after it.
