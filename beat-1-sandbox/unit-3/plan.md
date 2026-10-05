# Plan: issue #62, `/health` reports Redis unhealthy because `settings.redis_host` does not exist

Issue: https://github.com/codepath/pathreview-ai301-fa26-s3/issues/62
Repro comment: https://github.com/codepath/pathreview-ai301-fa26-s3/issues/62#issuecomment-5864660997 (commit `2f4e82f`)

## Diagnosis

The Redis probe in `api/routes/health.py` (lines 46-51) builds its client
from `settings.redis_host` and `settings.redis_port`. `Settings` in
`core/config.py` defines neither field. It defines a single
`redis_url` (line 14, default `redis://localhost:6379/0`). Reading the
missing attribute raises `AttributeError`, and the probe's
`except Exception` (line 55) turns that into `"redis": "unhealthy"`.
The probe never reaches `r.ping()`, so Redis's real state is never
checked.

What my repro shows, quoted from my posted comment:

- `docker compose exec redis redis-cli ping` returned `PONG`, so Redis
  was up and reachable.
- `GET /health` returned HTTP 503 with `"redis":"unhealthy"` in the body.
- The API log for that request: `redis_health_check_failed
  error="'Settings' object has no attribute 'redis_host'"`.

Redis is reachable, and the failure is the `AttributeError` raised before
any connection is attempted, so the cause is the wrong settings
attribute in the probe, not Redis and not the network.

One thing the repro also shows: in the same request the Postgres probe
fails with `Textual SQL expression 'SELECT 1' should be explicitly
declared as text('SELECT 1')`. That is a different defect (#61). It means
`/health` will still return 503 after my fix on a stack that has the #61
bug, so the 503 is not my success signal. The `redis` field is.

## Scope

In scope (what I will change):

1. `api/routes/health.py`: build the Redis client from `settings.redis_url`
   instead of `settings.redis_host` / `settings.redis_port`.
2. `pyproject.toml`: remove `attr-defined` from the
   `[[tool.mypy.overrides]]` entry for `api.routes.health` (line 154),
   because that suppression exists for #62 and CONTRIBUTING.md says a fix
   removes it.
3. `tests/unit/test_health.py` (new file): a unit test for the Redis
   probe.

Out of scope (what I will not touch):

- The Postgres probe and its `"SELECT 1"` string (#61). The `call-overload`
  and `index` entries stay in the mypy override because they belong to #61.
- `core/config.py`. I am not adding `redis_host` / `redis_port` fields,
  because the issue says the probe should use the existing `redis_url`,
  and adding fields would create a second place that must agree with it.
- The vector DB probe, the `except Exception` handling, the response
  shape, and any other file that reads Redis settings.

## Files I will touch

- `api/routes/health.py` (edit, about 5 lines)
- `pyproject.toml` (edit, one entry in a list)
- `tests/unit/test_health.py` (new)

## Approach

1. In `api/routes/health.py`, replace the `redis.Redis(host=..., port=...,
   db=0, decode_responses=True)` call with
   `redis.Redis.from_url(settings.redis_url, decode_responses=True)`.
   The `/0` database number is already part of the default `redis_url`,
   so dropping `db=0` keeps behavior the same. Leave `r.ping()` and the
   healthy/unhealthy handling as they are.
2. In `pyproject.toml`, change `disable_error_code = ["attr-defined",
   "call-overload", "index"]` for `api.routes.health` to
   `["call-overload", "index"]`.
3. Add `tests/unit/test_health.py`. It calls the `health_check` function
   directly with a fake `db` object whose async `execute` succeeds, and
   monkeypatches `redis.Redis.from_url` to return a fake client:
   - healthy case: the fake's `ping()` returns True. Assert
     `dependencies["redis"] == "healthy"` and that `from_url` was
     called with `settings.redis_url`.
   - unreachable case: the fake's `ping()` raises
     `redis.ConnectionError`. Assert the result is `"unhealthy"`, so the
     fix does not turn the probe into one that always says healthy.
4. Run `make lint`, `make typecheck`, `make test-unit`.

## Test plan

Automated: `pytest tests/unit/test_health.py`.

- Before the fix: the healthy-case test fails with `AttributeError:
  'Settings' object has no attribute 'redis_host'` (raised inside the
  probe and caught, so the assertion sees `"unhealthy"`).
- After the fix: both tests pass.

My unit 2 repro re-run, against the real stack (`cp .env.example .env`,
`docker compose up -d`, `make setup`, start the API, then the same
`curl http://localhost:8000/health` as in my repro comment):

- Before (from my repro): `"redis":"unhealthy"` in the body and
  `redis_health_check_failed ... 'Settings' object has no attribute
  'redis_host'` in the log.
- Expected after: `"redis":"healthy"` in the body, no
  `redis_health_check_failed` line in the log, and
  `redis_health_check_passed` at debug level. The overall status may
  still be 503 with `"postgres":"unhealthy"` while #61 is unfixed, and
  that is not a failure of this fix.

Also: `make typecheck` passes with `attr-defined` removed from the
override.

## Risks and unknowns

- **No existing test for #62.** I searched `tests/` for `xfail` and for
  `health` and found no seeded test for this issue, even though
  CONTRIBUTING.md and the PR template describe removing an xfail marker.
  So I am adding a new test instead. If a maintainer says there is a
  seeded test somewhere else, I will use that and remove its marker.
- **Removing `attr-defined` might expose other errors.** The override for
  `api.routes.health` also covers #61, and I have not run mypy without
  `attr-defined` yet. If mypy reports a different `attr-defined` error in
  that file, I will leave the entry alone and note it, because the
  issue's acceptance criteria and CONTRIBUTING.md only ask me to drop it
  for #62.
- **`from_url` with a non-default `REDIS_URL`.** I have only run with the
  default `redis://localhost:6379/0`. A URL with a password or a `rediss://`
  scheme should work with `from_url`, but I have not tested one.
- **Overall 503 stays until #61 is fixed.** Covered above.

## Deviations

The build matched the plan: same three files, same one-line change
(`redis.Redis.from_url(settings.redis_url, decode_responses=True)`), same
two tests. Two things in my "Risks and unknowns" section are now settled:

- Removing `attr-defined` from the `api.routes.health` mypy override did
  not expose any other error. `mypy` on the changed files reports
  "Success: no issues found", and the pre-commit mypy hook passed.
- There was still no seeded xfail test for #62 anywhere in `tests/`, so
  there was no marker to remove, and the new `tests/unit/test_health.py`
  stands on its own.

My test plan also said to re-run the `curl /health` repro on the real
Docker stack. Docker Desktop was not running during the build, so I did
that re-run afterward, on the same branch (`7426e83`). Redis answered
`PONG`, and `/health` returned `"redis":"healthy"` with
`redis_health_check_passed` in the log and no `redis_health_check_failed`
line. It still returned HTTP 503 because the Postgres probe fails with the
`text('SELECT 1')` error from #61, which is what the plan said would
happen, so nothing deviated there. The only thing I have not tried is a
non-default `REDIS_URL`.
