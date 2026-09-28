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
