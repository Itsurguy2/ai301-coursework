# Rubric: is this reproduction package ready to post?

## Checks

| Check | Evidence | Pass condition | Weight |
|---|---|---|---|
| Environment recorded | The repro report's environment section | The report states the exact tool/service version and OS/platform used to reproduce, and either matches the issue's stated target version or explicitly explains the difference | required |
| Steps followable | The repro report's reproduction steps and input files | A stranger could run the listed commands in order and hit the same behavior without guessing anything that matters to the trigger: exact commands, and each input either pasted in full or described precisely enough that any input meeting the description triggers it (e.g. 'a valid dependencies list plus an unrecognized category: section'). Fail when a step depends on private code or config, an unstated platform option the issue depends on, or 'set it up appropriately' | required |
| Behavior matches the issue | The repro report's output/error excerpt, read against the issue's description of the bug, and the report's stated outcome | If the report says it reproduced the bug: the shown artifact contains the issue's own specific symptom (same error message/type, same wrong value, same crash), not a different failure from altered input, a typo, or a different version. If the report says it could NOT reproduce: pass when it ran the issue's own input and steps, shows the artifact it got instead, and names what differed from the reporter's setup; that is an honest miss, not a wrong target. Fail when the artifact shows a different failure but the words call it the reported bug | required |
| Honest outcome | The repro report's stated conclusion, read against its own shown artifacts | The stated outcome (reproduced / could not reproduce / partially reproduced) is exactly what the shown artifacts support; an evidenced "could not reproduce" passes, a reproduction claim the artifacts don't back up fails | required |
| Claim promises, not asserts | The claim comment, read against the repro report in the same package | The comment names the specific issue and a concrete next step (investigate a named code path, report back). Fail if it promises a fix, a PR, or a date, or is interchangeable assign-me boilerplate. Stating a reproduction outcome (reproduced OR could not reproduce) is fine when the repro report in the same package shows it; only a claim-only draft, with no report yet, must promise the report instead of stating a result | required |
| Conventions and disclosure | The repo-facts block's stated contribution policy / AI policy / bug-report template, read against the claim comment and repro report text | Treat every package as AI-assisted work. If the stated policy explicitly requires DISCLOSING AI use, the comment must say so plainly or this fails. A policy that only requires comments be written by humans in their own words (no disclosure ask) passes when the comments read as specific, first-person, non-boilerplate writing; unverifiable authorship is not grounds for unclear. A repo with no stated policy passes. Where the bug-report template names required fields, the report supplies them with real content | required |
| Specific, not boilerplate | The claim comment and repro report text | The writing names concrete specifics (version numbers, exact error text, file/line references) rather than generic filler phrasing | preferred |

## Verdict rule

Full package: accept (ready) only if every required check passes; any
required check graded `fail` or `unclear` rejects it (hold). Claim-only
draft: accept only if every required check whose evidence is the claim
comment or repo conventions passes ("Claim promises, not asserts" and
"Conventions and disclosure"); checks that need the repro report are
marked `unclear` / not yet applicable per SKILL.md's claim-only rule and
are excluded from this verdict. Preferred checks never change either
verdict.
