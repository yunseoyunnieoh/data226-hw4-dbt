# DATA 226 Homework #4: dbt + Snowflake

dbt project that migrates the `session_summary` CTAS pipeline (from Homework #3 /
Week 5 lecture) into dbt models, snapshots, and tests. This is dbt only — it is
not run as an Airflow DAG.

## Pipeline

- **Source tables** (`raw.user_session_channel`, `raw.session_timestamp`): loaded
  from `s3://s3-geospatial/readonly/` per the Week 5 `build_elt_with_ctas.py`
  example.
- **Transform models** (`models/transform/`): `user_session_channel.sql`,
  `session_timestamp.sql` — cleaned pass-through of the raw tables, materialized
  as `ephemeral` (built as CTEs, no physical table/view).
- **Analytics model** (`models/analytics/session_summary.sql`): joins the two
  transform models on `sessionId`, materialized as a `table`.
- **Snapshot** (`snapshots/snapshot_session_summary.sql`): SCD Type 2 timestamp
  snapshot of `session_summary`, keyed on `sessionId`, tracking changes via `ts`.
- **Tests** (`models/schema.yml`): `unique` and `not_null` on `session_summary.sessionId`.

## Setup

```bash
python3 -m venv venv
source venv/bin/activate
pip install dbt-core dbt-snowflake
```

Configure `~/.dbt/profiles.yml` with a `hw4_project` profile using Snowflake
key-pair authentication (see `dbt_project.yml` for the profile name). Do not
commit `profiles.yml` — it is excluded via `.gitignore`.

## Commands

```bash
dbt debug     # verify Snowflake connection
dbt run       # build session_summary (transform models compile as CTEs)
dbt snapshot  # build snapshot.snapshot_session_summary
dbt test      # run unique + not_null tests on sessionId
```

## Reference

- Course: SJSU DATA 226, Week 5 — ELT & dbt (Keeyong Han)
