# Unit 3: Plan and Implement

Path: `beat-1-sandbox/unit-3/plan-and-implement.md`

---

## Your identity upstream

**GitHub username**

Itsurguy2

---

## Posted upstream

**Plan comment**

https://github.com/codepath/pathreview-ai301-fa26-s3/issues/62#issuecomment-5987860084

Plan for #62, built from my own repro (comment above, commit `2f4e82f`).

**Cause.** The Redis probe in `api/routes/health.py` builds its client from `settings.redis_host` and `settings.redis_port`. `Settings` only defines `redis_url`, so the probe raises `AttributeError` before it ever calls `ping()`, and the `except Exception` reports "unhealthy". In my repro, `redis-cli ping` returned `PONG` while `/health` showed `"redis":"unhealthy"` and the log said `'Settings' object has no attribute 'redis_host'`.

**Change.** One change, in `api/routes/health.py`: build the client with `redis.Redis.from_url(settings.redis_url, decode_responses=True)`. I'll also remove `attr-defined` from the `api.routes.health` mypy override in `pyproject.toml`, since CONTRIBUTING says a fix for #62 drops it. I won't touch the Postgres probe (#61, so `call-overload` and `index` stay in the override), `core/config.py`, or the vector DB probe.

**Tests.** A new `tests/unit/test_health.py` with a healthy case (fake client's `ping()` succeeds, expect `"redis":"healthy"` and `from_url` called with `settings.redis_url`) and an unreachable case (`ping()` raises, expect `"unhealthy"`). I'll also re-run my repro `curl` against the real stack: I expect `"redis":"healthy"` and no `redis_health_check_failed` in the log. The overall status may still be 503 because of the Postgres failure in #61, so I'm judging this fix by the `redis` field.

**Not checked yet.** I couldn't find a seeded xfail test for #62 in `tests/` (CONTRIBUTING mentions one), so I'm adding a new test. If one exists somewhere I missed, tell me and I'll use it and remove its marker. I haven't run mypy without `attr-defined` yet, and I've only tried the default `redis://localhost:6379/0`.

Several others are working this issue too; this plan is built from my own repro above.

---

## Implementation

**Branch**

fix/62-redis-health-url

**Evidence**

Before the fix (`pytest tests/unit/test_health.py -v` on the branch with the test added and `health.py` unchanged):

```
$ python -m pytest tests/unit/test_health.py -v
>       assert deps["redis"] == "healthy"
E       AssertionError: assert 'unhealthy' == 'healthy'
tests\unit\test_health.py:42: AssertionError
---------------------------- Captured stdout call -----------------------------
[debug    ] postgres_health_check_passed
[error    ] redis_health_check_failed      error="'Settings' object has no attribute 'redis_host'"
[debug    ] vector_db_health_check_passed
FAILED tests/unit/test_health.py::TestRedisHealthProbe::test_reachable_redis_reports_healthy
=================== 1 failed, 1 passed, 1 warning in 14.69s ===================
```

After the fix (commit `7426e83`):

```
$ python -m pytest tests/unit/test_health.py -v
tests/unit/test_health.py::TestRedisHealthProbe::test_reachable_redis_reports_healthy PASSED [ 50%]
tests/unit/test_health.py::TestRedisHealthProbe::test_unreachable_redis_reports_unhealthy PASSED [100%]
======================== 2 passed, 1 warning in 1.11s =========================

$ python -m pytest tests/unit -q
377 passed, 53 xfailed, 2 warnings in 22.76s

$ mypy api/routes/health.py tests/unit/test_health.py     (attr-defined removed from the override)
Success: no issues found in 2 source files
```

The unit test runs the real `health_check` function and the real Redis probe line, with only the Redis client faked, and its "before" failure shows the same `'Settings' object has no attribute 'redis_host'` error as my unit 2 repro.

Real-stack repro, re-run on the fixed branch (`docker compose up -d`, API started with `uvicorn api.main:app`). My unit 2 repro comment is the before for this one (`"redis":"unhealthy"` and `redis_health_check_failed ... 'redis_host'`):

```
$ docker compose exec redis redis-cli ping
PONG
$ curl -s -o /tmp/health.json -w "HTTP %{http_code}\n" http://127.0.0.1:8000/health
HTTP 503
$ cat /tmp/health.json
{"detail":{"status":"unhealthy","dependencies":{"postgres":"unhealthy","redis":"healthy","vector_db":"healthy"},"safety_events_last_hour":0,"timestamp":"2026-10-05T04:09:29.671137"}}
```

API log for that request:

```
[error    ] postgres_health_check_failed   error="Textual SQL expression 'SELECT 1' should be explicitly declared as text('SELECT 1')"
[debug    ] redis_health_check_passed
[debug    ] vector_db_health_check_passed
```

`redis` is now `healthy` and there is no `redis_health_check_failed` line. The overall status is still 503 because of the Postgres probe (#61), which this change does not touch.

---

## Eval iterations

**Run history**

1. Smoke test (`--limit 3`, partial): pkg-01 and pkg-03 matched gold; pkg-02 crashed with a Windows `UnicodeEncodeError` in the harness's stdin write, not a grading result.
2. Partial re-run of pkg-02 with `PYTHONUTF8=1`: accept, matching gold.
3. Full run 1 (the confirming run, saved as `eval-run.txt`): 20/20. Every category matched: clear-accept 7/7, scope-creep 4/4, thread-convention 2/2, unbuildable 3/3, wrong-cause 4/4.

I did not revise the rubric after the first full run, so there was only one full run.

**Package analysis**

pkg-04 (fzf #4260, Windows `less` swallowing keys). Gold: reject. My skill: reject. The plan is documentation-only (man page note, README examples, FAQ entry for the `> /dev/tty` workaround). Its diagnosis is consistent with both control runs, its scope is bounded, it names where to change things, and its test re-runs the repro with the documented command, so five of my six checks passed. The check that rejected it was thread-and-conventions: in the thread, the owner (junegunn) pointed at `src/tui/light_windows.go` as the culprit, posted a patched test binary from commit 8916cbc, and asked the reporter to test it. The plan comment never mentions any of that and proposes documenting the workaround instead. The skill's note on that check was that it "never addresses that claimed fix, instead proposing documentation-only without engagement." Gold agrees, because a plan that is fine on its own can still be wrong to post when it ignores what the maintainer already asked for. A rubric that only read the plan against the repro evidence would have accepted this one.

**Check rationale**

"thread-and-conventions", as it reads in `tools/plan-check/rubric.md`:

> The comment responds to any explicit maintainer direction in the thread (a requested approach, a claimed fix, a request for testing) instead of ignoring it, and honors every stated repo convention, including any AI disclosure the policy requires. If the thread has no direction and the repo states no applicable rule, pass

My group's worksheet rubric had four checks (diagnosis, scope, an evidence match, an automated test) and nothing about the thread or the repo's conventions, so a plan like pkg-04's would have passed. I added this check because the eval README says the 2-package thread-convention category cannot be bought back on volume. The parenthetical lists the three kinds of direction I saw in pkg-04 so a grader looks for a requested approach, a claimed fix, or a request for testing instead of only for a comment that exists. The last sentence is there because some packages, like calib-01, have an empty thread and no applicable rule, and they should not fail a check that has nothing to check.

**Trade-offs**

This check only asks whether the comment responds to the direction, not whether it responds well. A comment that mentions the maintainer's request in one line and then does something else would pass, so it is looser than a check that required the plan to follow the maintainer. It also depends on the thread highlights being complete: if the maintainer's direction is not in the excerpt, the check passes a comment that ignored it. In live mode it relies on the skill reading the real thread. I made it required, which means a plan that is otherwise good is held for a missed convention, such as a disclosure line, even though the fix itself is sound. I accepted that because the category exists to catch exactly that.
