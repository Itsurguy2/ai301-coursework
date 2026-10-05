# Procedure: how this skill grades a plan package

## Read order

1. Read the issue first. Note in one sentence what the reporter says is
   broken and what they expected to happen instead.
2. Read the thread highlights next. Note anything a maintainer asked
   for, claimed, or ruled out. If it says "(no comments)", note that.
3. Read the repo facts. Note the contribution policy, any AI policy
   or disclosure rule, and what the bug template asks for.
4. Read the repro evidence before the plan. Write down what it
   actually shows: the steps, the observed result, and any control run
   (the same thing run with one change). A control run tells you what
   the cause is not, so keep it in mind when you read the plan's cause.
5. Now read the candidate plan. Find its stated cause, its change and
   scope lines, its approach, its test, and any risks or unknowns.
6. Read the plan comment last. It is the part a maintainer would see.

Why this order: the plan has to be judged against the evidence, so the
evidence goes in your head first. If you read the plan first you will
tend to believe it.

## Evidence gathering

1. For diagnosis-matches-evidence: copy the plan's stated cause. Then
   copy the repro fact that supports it, or the repro fact that
   contradicts it. If the plan cites no repro fact, write "none cited".
2. For scope-bounded: list every file, area, or kind of change the
   plan says it will touch, and every thing it says it won't. Then
   next to each item write whether the diagnosed cause requires it.
3. For executable-by-stranger: find the sentence that says where the
   change goes (a file, function, or setting) and what changes there.
   If you cannot find one, write "no location".
4. For test-observes-outcome: copy the plan's test. Find the repro step
   it re-runs. Write down the result the test says you will see after
   the fix, and what you would see before it.
5. For thread-and-conventions: list each maintainer direction from the
   thread highlights and each stated repo rule from the repo facts.
   Then find where the plan comment answers each one. Write "not
   addressed" for any it skips.
6. For unknowns-stated-honestly: find any claim in the plan about
   behavior the repro did not test. Write whether it is called a
   guess or a thing to check, or stated flatly as true.

## Check execution

1. Run the checks in the order they appear in the rubric table.
2. For each check, use only the facts you gathered above and the
   pass condition written in the rubric. Do not add your own tests of
   quality. If the pass condition is met, grade pass, even if the plan
   is short or plain.
3. Grade fail when the evidence you gathered shows the pass condition
   is broken, and quote that evidence.
4. Grade unclear only when the package truly lacks what the check
   needs (for example, the plan has no test section at all, or the
   repro block is missing). Do not grade unclear just because you are
   unsure; look again first.
5. If you have already found a plain contradiction between the
   plan's cause and the repro evidence, you can grade the other checks
   without re-reading the whole package, but still grade every check.
6. Judge the plan itself. Do not give credit for headings, length,
   confident tone, or neat formatting, and do not take it away for
   their absence.

## Verdict assembly

1. List the grades of the five required checks.
2. If every required check is pass, the verdict is accept.
3. If any required check is fail, or unclear, the verdict is reject.
   Unclear counts as fail.
4. Ignore the preferred check when deciding. Still report its grade.
5. If the diagnosis check failed because the cause contradicts the
   repro evidence, the verdict is reject no matter what else passed.
6. In the output, quote the one fact that decided each check. For a
   reject, name the failing check first and quote what broke it.
7. End with the JSON block and nothing after it.
