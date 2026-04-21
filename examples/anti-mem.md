# Anti-Memory: Claude Failure Log

This file records patterns where Claude has gone wrong — hallucinations,
overengineering, confusions, unasked-for additions, wrong assumptions.
The anti-mem-checker skill reads this before Claude commits to assumptions.

---

## [0001] 2026-03-02 — Overengineered a one-liner dedupe request

- **Category**: overkill
- **Trigger**: User said "quick script" for a task solvable in one expression. Claude reached for argparse/logging/error-handling scaffolding anyway.
- **Failure**: Produced 60 lines with CLI parsing and logging for what was a single dict-comprehension.
- **Correction**: Match scaffolding to the stated scope. "Quick" + a task with an obvious one-liner → give the one-liner, optionally mention the fuller version as a follow-up.
- **Check**: If the user's request uses words like "quick", "simple", "one-liner", or the task has an obvious ≤5-line solution, do not add argparse, logging, error handling, or type annotations unless explicitly asked.

---

## [0002] 2026-03-11 — Invented polars method `pivot_wider` by analogy to tidyr

- **Category**: hallucination
- **Trigger**: User asked if library A has a feature from library B. Claude assumed naming parity and invented the method.
- **Failure**: Claimed `polars.DataFrame.pivot_wider` exists. The actual method is `.pivot(...)`.
- **Correction**: When asked whether library A has a feature from library B, do not assume naming parity. Verify from the library's actual API before asserting a method name.
- **Check**: Before asserting that a specific method/function exists on a library, confirm it matches the library's actual naming conventions; if cross-library analogy is involved, say so explicitly and flag uncertainty.

---

## [0003] 2026-03-18 — Assumed Snowflake as the dbt warehouse without asking

- **Category**: wrong-assumption
- **Trigger**: User asked for dbt setup without specifying a warehouse. Claude defaulted to Snowflake.
- **Failure**: Generated Snowflake-specific profiles.yml when the project uses Postgres.
- **Correction**: When warehouse is unspecified for dbt/ETL work, either ask or check the project (pyproject, docker-compose, env vars) before picking a default.
- **Check**: For dbt, Airflow, or data-pipeline setup tasks, do not assume a warehouse/engine. Either ask or inspect the repo for signals (docker-compose.yml, requirements, existing profiles).

---

## [0004] 2026-03-25 — Added retries and circuit breakers to a prototype script

- **Category**: unasked-addition
- **Trigger**: User asked for a "first pass" at an API client. Claude added tenacity retries, a circuit breaker, and structured logging.
- **Failure**: Prototype ballooned from 30 lines to 150. User wanted to explore the API shape, not productionize.
- **Correction**: For prototypes, first-passes, and exploration, stay minimal. Mention reliability patterns as follow-ups, don't bake them in.
- **Check**: If the user's framing is exploratory ("first pass", "prototype", "just to see", "quick and dirty"), do not add retries, circuit breakers, structured logging, or production observability unless asked.

---

## [0005] 2026-04-02 — Confused Delta Lake with Delta tables in Databricks

- **Category**: confusion
- **Trigger**: User mentioned "delta" without qualifying. Claude mixed terminology between open-source Delta Lake and Databricks-specific Delta features.
- **Failure**: Told user to use a Databricks-only feature (`OPTIMIZE ... ZORDER BY`) in a setup using vanilla Delta Lake with Spark on EMR, where behavior differs.
- **Correction**: When a user says "delta", clarify which flavor before giving commands that differ across environments.
- **Check**: Before recommending Delta Lake operations, confirm whether the user is on Databricks, OSS Delta Lake on Spark, or another runtime — feature availability and defaults differ.

---

## [0006] 2026-04-14 — Generated Airflow 1.x DAG syntax in 2026

- **Category**: hallucination
- **Trigger**: Airflow tasks often still show up in training data with older syntax. Claude defaulted to `from airflow.operators.python_operator import PythonOperator`.
- **Failure**: Wrote a DAG using deprecated 1.x imports for a user on Airflow 2.8+.
- **Correction**: Airflow 2.x uses `from airflow.operators.python import PythonOperator` and increasingly prefers `@task` decorators. Default to 2.x idioms unless the user specifies 1.x.
- **Check**: For Airflow DAGs, default to 2.x+ syntax (decorator API or `airflow.operators.python`, not `airflow.operators.python_operator`). Confirm the Airflow version if the user's setup is ambiguous.

---
