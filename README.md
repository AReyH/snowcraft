# snowforge

> The Snowflake standard library that never shipped.

[![CI](https://github.com/AReyH/snowforge/actions/workflows/ci.yml/badge.svg)](https://github.com/AReyH/snowforge/actions/workflows/ci.yml)
[![PyPI](https://img.shields.io/pypi/v/snowforge)](https://pypi.org/project/snowforge/)
[![Python](https://img.shields.io/pypi/pyversions/snowforge)](https://pypi.org/project/snowforge/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

`snowforge` is a Python toolkit that wraps the most common, tedious Snowflake
operations into clean, tested, pip-installable functions. Think of it as the
standard library that `snowflake-connector-python` never shipped.

---

## Install

```bash
pip install snowforge
```

Python 3.10+ required.

---

## What's in the box

| Module | What it does |
| --------- | ------------------------------------------------------------ |
| `connection` | Context-managed Snowflake connection with env var fallback |
| `merge` | Programmatic `MERGE INTO` builder — inspect the SQL before running it |
| `schema` | Schema introspection and breaking-change detection |
| `profiler` | Find expensive queries and runaway warehouses |
| `scd` | SCD Type 1 and Type 2 dimension helpers |

---

## Quick start

```python
from snowforge import SnowforgeConnection, MergeBuilder

with SnowforgeConnection() as conn:       # reads SNOWFLAKE_* env vars
    result = MergeBuilder(
        conn=conn,
        target_table="MYDB.PUBLIC.ORDERS",
        source_query="SELECT order_id, status, updated_at FROM MYDB.STAGING.ORDERS",
        match_keys=["order_id"],
    ).execute()

    print(f"Inserted: {result.rows_inserted}, Updated: {result.rows_updated}")
```

**Schema diffing:**

```python
from snowforge import SchemaInspector

diff = SchemaInspector(conn).diff("MYDB.STAGING.ORDERS", "MYDB.PUBLIC.ORDERS")
if diff.is_breaking:
    print(diff.to_markdown())    # ready to paste into a GitHub PR comment
```

**Query profiling:**

```python
from snowforge import QueryProfiler

for q in QueryProfiler(conn).top_expensive(n=10, lookback_hours=24):
    for hint in q.optimization_hints:
        print(f"  {q.query_id}: {hint}")
```

**SCD Type 2:**

```python
from snowforge import SCDManager

SCDManager(
    conn=conn,
    target_table="MYDB.DW.DIM_CUSTOMER",
    source_query="SELECT customer_id, name, email FROM MYDB.STAGING.CUSTOMERS",
    business_keys=["customer_id"],
    tracked_columns=["name", "email"],
).apply_type2()
```

---

## Development

```bash
git clone https://github.com/AReyH/snowforge.git
cd snowforge
python -m venv .venv && source .venv/bin/activate
pip install -e ".[dev]"

ruff check . && ruff format .
mypy snowforge/
pytest tests/unit/ -v --cov=snowforge --cov-report=term-missing
```

See [CLAUDE.md](CLAUDE.md) for the full project spec and contribution guide.

---

## License

MIT — see [LICENSE](LICENSE).
