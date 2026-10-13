# Evidence

Two kinds of evidence live here:

| Folder / file | Produced where | Status |
|---|---|---|
| `dev-sandbox/` | Development verification run (Linux, Python 3.13, real PostgreSQL 16, **no Docker/Airflow, CSV layer fallback**, test-only psql-backed DB driver) | committed: proves the logic end to end |
| `docker/` + `failure_injection_docker.*` | **Your** Docker Desktop stack on Windows, with Airflow | to capture: the authoritative evidence for grading |

> Note on `dev-sandbox/postgres_audit_queries.txt`: the `raw_pollutant_codes_known` rows marked *failed* (severity **warn**) are real history. They led to the YAML "Norway problem" fix (unquoted `NO` parsed as `False`; commit `fix: quote NO pollutant code…`). Runs after the fix pass that check.

## What is already in `dev-sandbox/`

| File | Shows |
|---|---|
| `metrics_comparison_second_run.json` | reproduced vs baseline, tolerance verdict `pass` (A16, A17) |
| `metrics_second_run.json`, `predictions_sample.csv` | regenerated metrics and predictions |
| `run_summary_second_run.json` | run record: per-stage status, durations, counts, layer fingerprints |
| `incremental_runs.md` | checkpoint 0 → 20,000 → 40,000 → 54,255, then a no-op run (A7) |
| `postgres_audit_queries.txt` | duplicate-key view = 0 after repeated runs (A9, A15); load log 0 inserted / 0 updated on reruns; partition-scoped load; run overview incl. failed runs |
| `failure_injection_dev-sandbox.md/.json` | four injected failures, protection checks, recovery (A13, A14) |
| `storage_benchmark.md/.json` | CSV / JSON / PostgreSQL (Parquet skipped: no pyarrow in the sandbox) |
| `pytest_output.txt` | 35 passed, 2 skipped (Airflow and pyarrow not installable in the sandbox) |

## Checklist: capture these on your machine (demonstration sequence §10.1)

Save screenshots as `docs/evidence/docker/NN_name.png`. Save command output as `.txt` with `... | Out-File docs\evidence\docker\NN_name.txt`.

| # | Lab step | Command / action | Save as |
|---|---|---|---|
| 1 | Repo + commit history | GitHub repo page; `git log --oneline` | `01_git_log.txt`, `01_github.png` |
| 2 | No secrets / large artifacts committed | `git ls-files \| Select-String -Pattern '\.env$\|\.csv$\|\.parquet$'` (should print only `.env.example`, `predictions_sample.csv`); `git check-ignore -v data/source/world_air_quality.csv .env` | `02_no_secrets.txt` |
| 3 | Checksum vs manifest | `dc run --rm pipeline python -m pipeline.cli verify-source` | `03_verify_source.txt` |
| 4 | Build/start environment | `dc up -d --build`; `dc ps` | `04_compose_ps.txt` |
| 5 | Run the pipeline via Airflow | Trigger DAG (mode=full, seed 42): **Graph view all green** | `05_dag_graph_success.png` |
| 6 | Raw / staging / curated / outputs | `Get-ChildItem -Recurse data\raw\committed, data\staging, data\curated, data\features, data\outputs\latest` | `06_layers.txt` |
| 7 | PostgreSQL tables + audit | `dc exec warehouse psql -U pipeline -d airquality -c "\dt staging.*" -c "\dt curated.*" -c "\dt ml.*" -c "\dt audit.*" -c "SELECT * FROM audit.v_run_overview LIMIT 5" -c "SELECT * FROM audit.load_log ORDER BY loaded_at DESC LIMIT 6"` | `07_postgres.txt` |
| 8 | Data-quality + code tests | `dc run --rm pipeline python -m pytest -v` and `dc exec airflow-scheduler python -m pytest /opt/project/tests/integration/test_dag_integrity.py -p no:cacheprovider` | `08_pytest.txt`, `08_dag_tests.txt` |
| 9 | Airflow dependency graph + run states | Graph view; Grid view showing runs; one task log showing JSON log lines | `09_graph.png`, `09_grid.png`, `09_task_log.png` |
| 10 | Second run, no duplicates | Trigger again; then `dc exec warehouse psql -U pipeline -d airquality -c "SELECT * FROM audit.v_duplicate_business_keys" -c "SELECT pipeline_run_id, table_name, inserted, updated, deleted, unchanged FROM audit.load_log ORDER BY loaded_at DESC LIMIT 6"` | `10_second_run.txt` |
| 11 | Baseline vs new metrics side by side | `dc run --rm pipeline python -c "import json;d=json.load(open('data/outputs/latest/metrics_comparison.json'));[print(r['model'],r['metric'],r['baseline'],r['reproduced'],r['abs_diff'],r['classification']) for r in d['metric_rows']];print(d['status'],d['outcome'])"` | `11_metrics_side_by_side.txt` |
| 12 | Failure / recovery | (a) Airflow: run with `source_override=data/tmp/broken_missing_pollutant.csv` → red `validate_raw`, downstream `upstream_failed`; then a clean run. (b) `dc stop warehouse` → `load_postgres` retrying/failing → `dc start warehouse` → Clear → green. (c) `dc run --rm -e EVIDENCE_LABEL=docker pipeline python scripts/failure_injection.py` | `12a_failed_run.png`, `12a_recovered.png`, `12b_retry.png`, `12b_cleared.png`, `failure_injection_docker.md` |
| 13 | Design decision to change for production | See README §"What I would change for production" | — |
| — | Storage benchmark with Parquet | `dc run --rm pipeline python -m pipeline.cli benchmark --repeats 3`; paste the table into `docs/storage_benchmark.md` §A | `benchmark_docker.md` |
| — | Partition demo | `dc run --rm pipeline python -m pipeline.cli partition --value Japan --load-db` | `partition_docker.txt` |
| — | Incremental demo | `dc run --rm pipeline python -m pipeline.cli reset-local --yes`, then trigger the DAG 4× with `mode=incremental`; `Get-Content data\checkpoints\ingest_checkpoint.json` | `incremental_docker.txt` |
