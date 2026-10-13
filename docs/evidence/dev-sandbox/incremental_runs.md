# Incremental ingestion evidence (dev sandbox)

Batch size 20,000 source rows; run 4 shows the idempotent no-op once the checkpoint reaches the end of the file. Generated from `data/outputs/runs/inc*/run_summary.json`.

| Run | Mode | New raw rows | Source rows | Checkpoint before → after | raw | staging | curated | Repro status |
|---|---|---|---|---|---|---|---|---|
| inc1 | incremental | 20000 | 1–20000 | None → 20000 | 20000 | 11706 | 6877 | not_applicable |
| inc2 | incremental | 20000 | 20001–40000 | 20000 → 40000 | 40000 | 22754 | 9973 | not_applicable |
| inc3 | incremental | 14255 | 40001–54255 | 40000 → 54255 | 54255 | 30545 | 11558 | pass |
| inc4 | incremental | 0 | None–None | 54255 → 54255 | 54255 | 30545 | 11558 | pass |
