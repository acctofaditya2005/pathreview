## Week 7 — Issue selection

**Issue link:** https://github.com/ascherj/pathreview/issues/154

**Issue title:** Health check DB probe passes a raw SQL string, which fails under SQLAlchemy 2.x

**Tier:** [x] Tier 1  [ ] Tier 2  [ ] Tier 3

**Problem summary:**
The database health check in `api/routes/health.py` calls `await db.execute("SELECT 1")` with a plain Python string. SQLAlchemy 2.x removed support for bare string SQL — it now requires all textual queries to be wrapped with `sqlalchemy.text()`. As a result, the `/health` endpoint always reports the database as unreachable and raises the error: "Textual SQL expression 'SELECT 1' should be explicitly declared as text('SELECT 1')". The fix is to import `text` from `sqlalchemy` and change the call to `await db.execute(text("SELECT 1"))` so the probe works correctly under SQLAlchemy 2.x.

**Branch name:** fix/154-health-check-sqlalchemy-text

**Setup confirmation:** [x] App runs locally at localhost:5173

**Cohort ledger:** [ ] Issue added to cohort ledger
