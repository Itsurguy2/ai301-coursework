# Unit 2 — Claim and Reproduce

Path: `beat-1-sandbox/unit-2/reproduction.md`

---

## Your identity upstream

**GitHub username**

Itsurguy2

---

## Posted upstream

**Claim comment**

https://github.com/codepath/pathreview-ai301-fa26-s3/issues/62#issuecomment-5864553725

Hi, I'm working through this repo as a course assignment and would like to claim this issue. I'm going to reproduce the `AttributeError` on `settings.redis_host` in `api/routes/health.py` and report back with what I find, including my environment and the exact output.

**Reproduction comment**

https://github.com/codepath/pathreview-ai301-fa26-s3/issues/62#issuecomment-5864660997

Reproduced on commit `2f4e82f52efbcfcc57d65b3fa5348672163ca088` (current `main` of this repo).

**Environment**
- Windows 11 (build 10.0.26200), Git Bash
- Python 3.11.15 (the project `.venv` created by `make setup`)
- Docker 29.8.0, Docker Compose v5.5.1; Redis server 7.4.9 from the repo's `docker-compose.yml`
- `redis` Python client 8.1.0

**Steps** (from `docs/SETUP.md`)

```bash
cp .env.example .env              # defaults, REDIS_URL=redis://localhost:6379/0
docker compose up -d              # db, redis, vector-db all report healthy
make setup                        # venv + deps; migrations 001 -> 002 applied
.venv/Scripts/uvicorn api.main:app --host 0.0.0.0 --port 8000   # backend half of `make run`
```

Two Windows deviations: I ran the backend half of `make run` directly because the frontend isn't needed to call `/health`, and I re-ran `scripts/seed_db.py` with `PYTHONUTF8=1` because it crashed printing a `✗` to the Windows console. Neither changes the health probe.

Confirmed Redis is reachable:

```
$ docker compose exec redis redis-cli ping
PONG
```

Called the health endpoint:

```
$ curl -s -o /tmp/health.json -w "HTTP %{http_code}\n" http://localhost:8000/health
HTTP 503
$ cat /tmp/health.json
{"detail":{"status":"unhealthy","dependencies":{"postgres":"unhealthy","redis":"unhealthy","vector_db":"healthy"},"safety_events_last_hour":0,"timestamp":"2026-09-28T06:21:17.924477"}}
```

API log for that request:

```
[error    ] postgres_health_check_failed   error="Textual SQL expression 'SELECT 1' should be explicitly declared as text('SELECT 1')"
[error    ] redis_health_check_failed      error="'Settings' object has no attribute 'redis_host'"
[debug    ] vector_db_health_check_passed
```

**Observed:** Redis answers `PONG`, but `/health` reports `"redis": "unhealthy"`, and the log shows the `AttributeError` for `redis_host`. That's the behavior this issue describes: the probe in `api/routes/health.py` reads `settings.redis_host` / `settings.redis_port`, `Settings` in `core/config.py` only defines `redis_url`, and the `except Exception` turns the `AttributeError` into "unhealthy".

**Caveat:** the Postgres probe fails in the same request for an unrelated reason (the raw `"SELECT 1"` string, which is #61), so the 503 on its own isn't evidence for this bug. The Redis part is shown independently by the `PONG` plus the `redis_host` error in the log.

Next, I'll look at switching the probe to build its client from `settings.redis_url`.

## Eval iterations

**Run history**

1. Full run 1 (00:55 EDT): 17/20, is below the bar. clear-accept 5/8; every
   reject category matched. All three misses were false rejects:
   pkg-09 and pkg-10 (honest cannot-reproduce reports) so failed "Behavior
   matches the issue", and pkg-03 got `unclear` on "Conventions and
   disclosure" because the grader couldn't verify human authorship.
2. Revision A: "Behavior matches" now splits on the stated outcome (a
   claimed repro must show the issue's own symptom; an evidenced
   could-not-reproduce passes). "Conventions" now separates a
   disclosure requirement from a human-voice requirement. Both loosen
   checks, so the --only run added canaries: pkg-20 (disclosure),
   pkg-02/pkg-08 (wrong-target), pkg-14 (no-evidence), plus calib-03.
   Result: 7/7 scored, calib-03 still rejected.
3. Full run 2: 17/20 again, with different misses. pkg-01 and pkg-09
   failed "Claim promises, not asserts" for stating a result the report
   backed up, and pkg-05 failed "Steps followable" for describing env.yml
   instead of pasting it. These packages passed in run 1, so the wording
   was ambiguous enough that the grader read it differently run to run.
4. Revision B: the claim check allows stating an the outcome that the
   package's own report shows (only a claim-only draft must promise).
   The steps check accepts an input described precisely enough to
   trigger the behavior. Canaries: pkg-19 and pkg-13 (boilerplate or
   over-promising claims), pkg-18 and pkg-06 (unfollowable), and pkg-20.
   Result: 8/8.
5. Full run 3 (confirming, saved as eval-run.txt): 20/20, every
   category matched.

**Package analysis**

pkg-09 (fd, argument-size flush reordering). Gold: accept. My rubric:
reject in run 1, accept in run 3. The report tries to trigger the issue's
scenario 2, shows a marker log in correct order, and says plainly it
could not reproduce, naming what likely differed (uniform file-name
lengths, a 2 MiB ARG_MAX). My original "Behavior matches the issue"
check required the artifact to show the issue's symptom, so it treated
an honest miss like a wrong target. The gold label is right: an
evidenced cannot-reproduce tells the maintainer something true. The
fixed check now asks what the report *claims* and whether the artifact
supports it, which still fails calib-03 (claims the panic, shows a
syntax error) but passes pkg-09.

**Check rationale**

"Behavior matches the issue", as it now reads:

> If the report says it reproduced the bug: the shown artifact contains
> the issue's own specific symptom (same error message/type, same wrong
> value, same crash), not a different failure from altered input, a
> typo, or a different version. If the report says it could NOT
> reproduce: pass when it ran the issue's own input and steps, shows the
> artifact it got instead, and names what differed from the reporter's
> setup; that is an honest miss, not a wrong target. Fail when the
> artifact shows a different failure but the words call it the reported
> bug

This check comes from the rubric swap. On calib-03, three of the four
rubrics in my group said "ready" because they checked that the report
had an environment, steps, and expected/actual sections. Only a check
that reads the artifact against the issue catches `{ 1: {} }` producing
a parse error instead of `panic: not a string`.

**Trade-offs**

Loosening "Behavior matches" to pass honest cannot-reproduce reports
risks passing a lazy "couldn't reproduce" with a token attempt, so the
check still requires the issue's own input and steps, a shown artifact,
and named differences. Likewise, letting a claim comment state a result
the report backs up makes my eval rubric slightly more permissive than
my live-mode rule (a claim posted before reproducing must promise, not
assert). I kept that stricter rule in the claim-only branch of the check
and the verdict rule rather than dropping it.

---

Related paths: `eval-run.txt` in this directory; your skill's files in
`tools/repro-check/`.
