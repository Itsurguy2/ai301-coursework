# Rubric: is this a good first issue?

## Checks

| Check | Evidence | Pass condition | Weight |
|---|---|---|---|
| Maintainer alive | Repo facts: "last 5 default-branch commits" dates, and "maintainer first-response sample" (or, live mode, the commit list and recent issue replies from Owner/Member/Collaborator badges) | At least one default-branch commit within 90 days of the capture/today date, OR the maintainer first-response sample shows a reply from an Owner/Member/Collaborator within 14 days of an issue being opened | required |
| Repo in use | Repo facts: "archived:" flag, "latest release" date, "last push to any branch" date, star count (or the equivalent live-mode sidebar signals) | Not archived, AND (latest release within the last 12 months OR last push within the last 90 days OR stars >= 100) | required |
| Scope fits a newcomer | Issue body and comment thread text | Fails if the issue is an umbrella/tracking issue listing multiple sub-items, if the thread shows an unresolved design debate with no maintainer decision, if a maintainer states the fix touches core internals, or if the issue is a pure usage/support question rather than a concrete bug or feature ask. A terse body or missing repro steps does not by itself fail this check. | required |
| Nobody already on it | Repo facts: "this issue: assignees:", "linked PRs:" with state, and the Comments section for claim comments (or the live-mode Assignees box, Development box, and thread) | Fails if there is a current assignee, an open linked PR addressing the issue, or a claim comment ("I'll take this" / "working on this") within the last 14 days that went unanswered/unchallenged by a maintainer. A closed, unmerged linked PR (abandoned attempt) does not fail this check. | required |
| Contribution policy allows AI-assisted work | Repo facts: "contribution policy" line (or CONTRIBUTING.md / AI_POLICY.md / PR template on github.com) | Fails only on an outright ban on AI-generated/AI-assisted contributions. Disclosure, human-review, or testing conditions pass. Silence passes. | required |
| Good-first-issue label | Issue labels in the repo-facts block or issue sidebar | Issue carries a "good first issue" (or equivalent newcomer-friendly) label | preferred |
| Clear acceptance criteria | Issue body | Issue states concrete acceptance criteria or reproduction steps a newcomer could work from without further clarification | preferred |

## Verdict rule

Accept if every required check passes. Any required check graded `fail` or
`unclear` rejects the issue. Preferred checks never change the verdict; they
only rank issues that are already accepted, most preferred-passes first.
