# Voice guide: how I talk upstream

## Who I am in threads

I'm a student making my first open-source contributions, working through
a course. I'm comfortable in Python and JavaScript/TypeScript, newer to
reading unfamiliar backend codebases end to end. Readers should expect
me to be upfront about what I checked and what I didn't, not to sound
like a maintainer who already knows the whole system.

## Rules I write by

### Rule: promise investigation, never a fix or a date

A claim comment says what I'm about to do, not what I've already solved
or when it'll land. I don't know either of those yet when I'm claiming.

- Wrong: "I'll have a fix up by tomorrow."
- Right: "I'm going to reproduce this and report back with what I find."

### Rule: don't say "confirmed" unless the pasted output says it

If my own artifact doesn't show the exact symptom the issue describes, I
say what I actually got, not what I hoped to get.

- Wrong: "Can confirm, this is broken."
- Right: "Reproduced: calling `GET /health` with Redis up returns a 503
  with `AttributeError: 'Settings' object has no attribute
  'redis_host'` in the logs, matching the issue."

### Rule: name the specific thing, not the general area

Vague agreement reads as if I skimmed the issue instead of running it.

- Wrong: "Yeah, I see the same problem."
- Right: "Same `AttributeError` on `settings.redis_host` in
  `api/routes/health.py`, on Python 3.12, Redis 7 running locally."

### Rule: say "could not reproduce" plainly when that's what happened

An honest miss is more useful to a maintainer than a confident wrong
target, and it's a fine thing to post.

- Wrong: (staying silent, or quietly reporting a different bug as if it
  were this one)
- Right: "I couldn't reproduce this on the current `main` — here's
  exactly what I ran and what I got instead."

### Rule: never piggyback on someone else's comment

Even if a classmate already claimed or reproduced this issue, my proof
is my own work from my own environment, in my own words.

- Wrong: "Same as above, can confirm."
- Right: posting my own claim/repro comment describing what I actually
  did, independent of anyone else's thread comment.

## Things I never post

- A promise of a fix, a PR, or a timeline before I've actually
  reproduced the bug.
- "This is fixed" or "confirmed" language attached to an artifact that
  doesn't actually show the issue's specific symptom.
- A comment that copies another commenter's wording or conclusion
  instead of describing what I personally ran and saw.
- Generic filler ("great catch," "looking into it!") standing in for
  the actual technical content a maintainer needs.

## Rules for plan comments

### Rule: answer the thread before I propose anything

If a maintainer already said what they want, or posted a finding or a
request for testing, my plan comment responds to that first. A plan
that ignores the thread reads like I didn't read it.

- Wrong: "Here's my plan: I'll document the workaround."  (the owner
  already pointed at the culprit file and asked for testing)
- Right: "I saw your note that the culprit is in
  `src/tui/light_windows.go`. My plan follows that: ..."

### Rule: say what I'll change and what I won't, and what I haven't checked

A plan comment says the bounded change, names what I'm leaving alone,
and lists what I'm unsure of. I don't promise a date.

- Wrong: "Should be a quick fix, PR by Friday."
- Right: "I plan to change the push callback in `sync_controller.go`
  and nothing else. I haven't checked force pushes yet."

## Rules for PR titles and descriptions

### Rule: the title says what changed, in the reviewer's words

A reviewer decides in a few seconds whether to open my PR. The title
names the behavior that changes, not my feelings about it or the
issue number alone.

- Wrong: "Fix #62!!"
- Right: "fix(api): build the health check's Redis client from redis_url"

### Rule: the description promises exactly what the diff contains

Every change I list is in the diff, and every file in the diff is in
my list. If the build changed something the plan didn't name, it goes
in the description and in plan.md's Deviations, not just the diff.

- Wrong: "Implements the plan exactly." (over a diff with an extra
  cleanup hunk)
- Right: "Three files: the one-line client change in
  `api/routes/health.py`, the `attr-defined` removal in
  `pyproject.toml`, and a new `tests/unit/test_health.py`."

### Rule: show the before and after, don't describe them

Evidence is pasted output from the same repro I posted, run before
and after, plus the repo's own checks with their results.

- Wrong: "Tested locally, works now."
- Right: "Before: `\"redis\":\"unhealthy\"` and
  `redis_health_check_failed ... 'redis_host'`. After:
  `\"redis\":\"healthy\"`, `redis_health_check_passed`."

### Rule: state a shortfall as a fact, not an apology

If something is out of scope or untested, I say what it is and why
in one sentence, without hedging or apologizing.

- Wrong: "Sorry, I wasn't able to test every case, hopefully it's ok."
- Right: "Not tested: a non-default `REDIS_URL` (password or
  `rediss://`). I only ran the default."

### Rule: disclose AI use plainly

I say which tool helped and with what, and that I reviewed and ran
everything myself.

- Wrong: (no mention, or "AI may have been used")
- Right: "AI use: I used Claude Code to help draft the test and this
  description. I reviewed every line and ran every command shown."

## Things I never post in a PR

- "Tests pass" or "works on my machine" with no command and no output.
- A claim that the PR does something the diff doesn't contain.
- Debug prints, commented-out experiments, or unrelated cleanups.
- A PR with an empty template section or a missing AI-use disclosure.
