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
