# Unit 1: Issue selection

## Chosen issue

https://github.com/codepath/pathreview-ai301-fa26-s3/issues/62

"Health check references `settings.redis_host`, which does not exist on
Settings" — a single-file backend bug in `api/routes/health.py`: the Redis
probe reads `settings.redis_host`/`settings.redis_port`, which `Settings`
doesn't define (it only has `redis_url`), so the resulting `AttributeError`
gets swallowed by a broad `except Exception` and `/health` reports Redis as
down even when it's reachable.

## Skill's verdict on the chosen issue

Run in live mode against three good-first-issue candidates from the Path
Review repo (`codepath/pathreview-ai301-fa26-s3`); all three were accepted.
Verdict for the chosen issue (#62):

```json
{
  "item": "https://github.com/codepath/pathreview-ai301-fa26-s3/issues/62",
  "checks": [
    {"name": "Maintainer alive", "grade": "pass", "evidence": "Default-branch commits as recent as 2026-09-16, days before the run"},
    {"name": "Repo in use", "grade": "pass", "evidence": "Not archived; last push 2026-09-16 (within 90 days)"},
    {"name": "Scope fits a newcomer", "grade": "pass", "evidence": "Single bounded bug: wrong Settings field name in one file, with explicit repro steps"},
    {"name": "Nobody already on it", "grade": "pass", "evidence": "assignees: none; linked PRs: none; no claim comments on this issue"},
    {"name": "Contribution policy allows AI-assisted work", "grade": "pass", "evidence": "docs/CONTRIBUTING.md sets process rules (branch naming, Conventional Commits, green CI) but states no AI restriction"},
    {"name": "Good-first-issue label", "grade": "pass", "evidence": "labels include \"good first issue\""},
    {"name": "Clear acceptance criteria", "grade": "pass", "evidence": "Body gives an exact repro call, the expected vs. actual response, and the log signature to look for"}
  ],
  "verdict": "accept"
}
```

The other two accepted candidates, in rank order behind #62, were
[#53](https://github.com/codepath/pathreview-ai301-fa26-s3/issues/53) (PII
scrubber regex fix) and
[#73](https://github.com/codepath/pathreview-ai301-fa26-s3/issues/73)
(README/`.env.example` mismatch). #62 ranked first for fit: it's a
Python backend bug I can reason about end to end, and it teaches a
realistic lesson (an overly broad `except` hiding a real defect) without
needing specialized domain knowledge like the PII regex issue or being as
trivial as the doc-sync issue.

## Reflection

**Why this issue fits me:** I'm comfortable in Python and want more
practice reading someone else's backend code and tracing a bug from a
symptom (a misleading health-check response) back to its root cause (a
typo'd config field caught by an over-broad exception handler). It's
scoped to one file and one behavior, so I can hold the whole fix in my
head.

**What the rubric run taught me:** matching against the eval set exposed
that my "scope" check is too eager to reject issues that describe several
possible technical causes for one symptom, treating that the same as a
true multi-part tracking issue. See the issue analysis below.

**What I'd watch for going into Unit 2:** the CONTRIBUTING.md requires all
five CI jobs green, and this repo uses `strict=True` `xfail` markers on
tests for seeded bugs — my fix needs to also remove #62's marker/suppression
(a `mypy` `attr-defined` suppression is called out by issue number in
`pyproject.toml`), not just patch the code.

## Run history

1. First full run (20 issues) crashed partway through: `run_eval.py`
   piped each prompt to `claude -p` over stdin using Python's default
   locale encoding, which is `cp1252` on this Windows machine. Several
   eval bundles contain emoji/arrow characters, and encoding them for
   the child process's stdin raised `UnicodeEncodeError` in the writer
   thread; the orphaned `claude` process then timed out waiting for
   stdin ("no stdin data received"). 7 of 20 items errored, so the
   harness correctly refused to write `eval-run.txt` (partial/errored
   run).
2. Fixed the bug in `run_eval.py` by passing `encoding="utf-8",
   errors="replace"` to the `subprocess.run` call in `grade_one`
   (previously bare `text=True`, which falls back to the OS locale
   encoding). This only changes how the harness talks to the CLI over
   the pipe; it doesn't touch `rubric.md` or `SKILL.md`, the two files
   the run's provenance header fingerprints.
3. Re-ran the full 20-issue set with the fix: no errors, and the harness
   wrote `eval-run.txt` and `results.json`. Final score: 15/20 scored
   items agree with gold (below the 18/20 bar), category floor met
   (`claimed 4/4  clear-accept 5/8  dead-repo 3/3  policy 1/1  scope
   2/4` — at least one match in every category). This is the run
   committed as `eval-run.txt`.

## Issue analysis

`issue-19` (source: `zxcalc/zxlive#517`, "Selecting large subgraphs in
proof mode freezes the UI"): **gold label is `accept`; my rubric graded
it `reject`**, failing the required "Scope fits a newcomer" check (and,
consequently, the two preferred checks that depend on labels/acceptance
criteria being present).

My rubric's scope check reads: "Fails if the issue is an umbrella/tracking
issue listing multiple sub-items, if the thread shows an unresolved
design debate with no maintainer decision, if a maintainer states the fix
touches core internals, or if the issue is a pure usage/support question
rather than a concrete bug or feature ask." The issue body lists "two
potential causes" for the freeze and three "additional suggestions" for
fixing it, which is exactly the shape my check pattern-matches to an
umbrella issue with sub-items to split up. But re-reading it, that's not
what's happening: it's one bounded bug (the UI freezes when selecting
large subgraphs) with the reporter brainstorming several ways a
contributor *could* address it, not a maintainer asking for several
separate deliverables. The issue has no `good first issue` label and no
formal acceptance criteria either, which is why my preferred checks
missed it too — but the gold label treats "the freeze goes away" as
acceptance criteria enough on its own. My check conflated "the writeup
mentions multiple things" with "the work is multiple things," and that's
the gap.

## Check rationale

The check that sank `issue-19`, quoted exactly as it appears in the
`rubric.md` uploaded with this submission:

> Fails if the issue is an umbrella/tracking issue listing multiple
> sub-items, if the thread shows an unresolved design debate with no
> maintainer decision, if a maintainer states the fix touches core
> internals, or if the issue is a pure usage/support question rather
> than a concrete bug or feature ask. A terse body or missing repro
> steps does not by itself fail this check.

## Trade-offs

My rubric trades recall for precision on the scope family: treating "the
issue body lists multiple items" as a proxy for "the work is unscoped"
correctly rejects real tracking issues (helping the `dead-repo` and
`policy` categories stay clean) but also catches single bugs that are
merely *described* with a list of causes or approaches, which is most of
what cost me the `scope` (2/4) and `clear-accept` (5/8) categories. A
looser check — for example, only failing scope when the issue explicitly
says "split into sub-issues" or a maintainer asks for that — would
recover those accepts, but risks the opposite failure: accepting a true
umbrella issue that looked bounded on a skim. Given that a first
contribution costing a newcomer a maintainer's patience is worse than a
newcomer skipping one good issue, I'd rather my rubric under-accept on
this arguable boundary than over-accept, even though it costs points here.
