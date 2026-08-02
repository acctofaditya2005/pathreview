## Week 7 — Issue selection

**Issue link:** https://github.com/ascherj/pathreview/issues/154

**Issue title:** Health check DB probe passes a raw SQL string, which fails under SQLAlchemy 2.x

**Tier:** [x] Tier 1  [ ] Tier 2  [ ] Tier 3

**Problem summary:**
The database health check in `api/routes/health.py` calls `await db.execute("SELECT 1")` with a plain Python string. SQLAlchemy 2.x removed support for bare string SQL — it now requires all textual queries to be wrapped with `sqlalchemy.text()`. As a result, the `/health` endpoint always reports the database as unreachable and raises the error: "Textual SQL expression 'SELECT 1' should be explicitly declared as text('SELECT 1')". The fix is to import `text` from `sqlalchemy` and change the call to `await db.execute(text("SELECT 1"))` so the probe works correctly under SQLAlchemy 2.x.

**Branch name:** fix/154-health-check-sqlalchemy-text

**Setup confirmation:** [x] App runs locally at localhost:5173

**Cohort ledger:** [ ] Issue added to cohort ledger

---

## Week 8 — Reproduction & solution planning

**Reproduction commit link:** https://github.com/acctofaditya2005/pathreview/commit/f9227d7b8a546559dcc27aacf0c32316266ec743

**Reproduction summary:**
Added a failing unit test in `tests/unit/test_health.py` that mocks `db.execute` to raise the same `ArgumentError` SQLAlchemy 2.x raises for bare string SQL. Running the test against the unfixed `health.py` confirms the route catches the error and returns HTTP 503 with `postgres: "unhealthy"`, reproducing the bug exactly as described in issue #154.

**PLAN.md link:** https://github.com/acctofaditya2005/pathreview/blob/fix/154-health-check-sqlalchemy-text/PLAN.md

**Walkthrough video (recommended):** N/A

**Blockers or open questions:**
None — root cause is clear and the one-line fix is straightforward. Will grep the full codebase for other raw `db.execute(` string calls before opening the PR.

---

## Week 9 — Solution building & PR submission

### Check-in 1 (mid-week)

**Current progress:**
Implemented the fix from PLAN.md steps 1–2: added `from sqlalchemy import text` to `api/routes/health.py` and changed `await db.execute("SELECT 1")` to `await db.execute(text("SELECT 1"))`. Steps 3–4 (failing/passing tests) were already completed in Week 8 in `tests/unit/test_health.py`. Confirmed via grep that this is the only raw-string `db.execute(` call in the codebase, so no other files need the same fix.

**Next steps:**
Run `make check` and `make test-unit` to confirm the fix passes both new tests and introduces no regressions (step 5 of PLAN.md), then open the PR against `ascherj/pathreview` and request a peer/mentor review before marking it ready.

**Blockers:**
None.

---

### Check-in 2 (end of week)

**PR link:** https://github.com/ascherj/pathreview/pull/565 (currently open as a draft, pending peer/mentor review before marking ready)

**Branch:** `fix/154-health-check-sqlalchemy-text`

**What you built:**
Wrapped the raw `"SELECT 1"` string in `sqlalchemy.text()` in the `/health` route's Postgres probe (`api/routes/health.py`), since SQLAlchemy 2.x rejects bare string SQL and was causing the health check to always report the database as unhealthy (HTTP 503) even when it was reachable.

**Tests added or updated:**
`tests/unit/test_health.py` — added `test_health_check_db_probe_fails_with_raw_string` (mocks `db.execute` to raise the same `ArgumentError` SQLAlchemy 2.x raises for bare strings, confirming the pre-fix bug returns 503/`unhealthy`) and `test_health_check_db_probe_passes_with_text_wrapper` (mocks a successful `db.execute` and asserts it's called with a `text()`-wrapped clause rather than a plain string). Both are now marked `@pytest.mark.unit` so they run under `make test-unit`.

**Self-review confirmation:** [x] make check passes  [x] make test-unit passes
(Confirmed no *new* failures: ruff has 4 pre-existing issues in `health.py` and mypy has 11 pre-existing errors in `health.py`, both identical before and after this change — verified by diffing `ruff check`/`mypy` output with the fix stashed vs. applied. `make test-unit` has 53 pre-existing failures unrelated to this issue, unchanged by this PR; the 2 new tests in `test_health.py` pass.)

**Draft PR feedback received from:** none yet — requesting review in the cohort Slack channel before marking ready for review.
