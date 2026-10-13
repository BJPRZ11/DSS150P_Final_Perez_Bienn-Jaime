# Failure injection & recovery evidence (dev-sandbox)

_Generated 20261013T071320Z by `scripts/failure_injection.py`; raw data in `failure_injection_dev-sandbox.json`._

| Scenario | Failed at | Published output unchanged | Checkpoint not advanced | DB unchanged | Quarantined | Recovery | Verdict |
|---|---|---|---|---|---|---|---|
| schema | validate_raw | True | True | True | True | success (`fi_recovery_20261013T071204Z`) | **PASS** |
| invalid_row | validate_raw | True | True | True | True | success (`fi_recovery_20261013T071225Z`) | **PASS** |
| invalid_config | config load | True | True | True | False | success (`fi_recovery_20261013T071244Z`) | **PASS** |
| db_unavailable | load_postgres | True | True | True | False | success (`fi_db_unavailable_20261013T071301Z`) | **PASS** |

## Error messages captured

**schema** — Temporary copy of the source with the required column 'Pollutant' removed.

```
PIPELINE ERROR [DataQualityError]: Data quality failure in layer 'raw': raw_schema_required_columns, raw_critical_not_null, raw_source_checksum_matches_manifest
```

**invalid_row** — Temporary copy with one unparseable concentration (row 101: Value='not_a_number').

```
PIPELINE ERROR [DataQualityError]: Data quality failure in layer 'raw': raw_value_parseable, raw_source_checksum_matches_manifest
```

**invalid_config** — Invalid configuration value PIPELINE_LAYER_FORMAT=xlsx (environment only).

```
PIPELINE ERROR [ConfigError]: storage.layer_format must be one of ['csv', 'parquet'], got 'xlsx'
```

**db_unavailable** — PostgreSQL unreachable (POSTGRES_PORT pointed at a closed port); then the SAME run is resumed from load_postgres once the database is reachable again.

```
PIPELINE ERROR [DatabaseError]: Cannot connect to PostgreSQL at 127.0.0.1:5999/airquality: psql terminated: psql: error: connection to server at "127.0.0.1", port 5999 failed: Connection refused | 	Is the server running on that host and accepting TCP/IP connections?
```
