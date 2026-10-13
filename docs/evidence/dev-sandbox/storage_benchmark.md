| Dataset | Format | Rows | Write (s) | Read (s) | Query (s) | Size (KB) | Same records | Max float err | Status |
|---|---|---|---|---|---|---|---|---|---|
| staging_readings | csv | 30,545 | 0.3021 | 0.082 | 0.0984 | 4,791 | True | 0.0e+00 | ok |
| staging_readings | json_lines | 30,545 | 0.3321 | 0.3249 | 0.2546 | 11,064 | True | 8.9e-16 | ok |
| staging_readings | parquet | 30,545 |  |  |  |  |  |  | skipped (`Import pyarrow` failed. pyarrow is required for parquet sup) |
| staging_readings | postgresql | 30,545 | 0.4338 | 0.3142 | 0.0049 | 5,192 | True | 0.0e+00 | ok |
| curated_station_day | csv | 11,558 | 0.0852 | 0.0273 | 0.0266 | 1,150 | True | 2.2e-16 | ok |
| curated_station_day | json_lines | 11,558 | 0.0485 | 0.0644 | 0.08 | 2,959 | True | 8.9e-16 | ok |
| curated_station_day | parquet | 11,558 |  |  |  |  |  |  | skipped (`Import pyarrow` failed. pyarrow is required for parquet sup) |
| curated_station_day | postgresql | 11,558 | 0.114 | 0.1159 | 0.0018 | 1,736 | True | 2.2e-16 | ok |

_Median of 3 repeats. Environment: python 3.13.16, numpy 2.5.3, pandas 3.0.5, scikit-learn 1.9.1._
