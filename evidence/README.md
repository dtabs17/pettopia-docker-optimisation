# Benchmark evidence

The `build-times.csv` file contains the ten measured wall-clock build times from the November 2026 Pettopia Docker optimisation experiment. These are historical measurements, not new test runs.

## How the times were collected

- Each observation is the `real` elapsed time, in seconds, of a completed `docker build --no-cache` command timed in WSL Ubuntu Bash.
- Five runs were recorded for the single-stage baseline, followed by five runs for the multi-stage configuration. The batches were sequential, not randomised.
- `run_in_batch` is the run number **within each configuration**. The same number in different configurations does not indicate a matched pair.
- Both configurations used the same laptop, network and application source, but background workload, download activity and thermal conditions were not continuously measured.

## Runs excluded from the statistical comparison

| Run | Elapsed time (s) | Reason |
| --- | ---: | --- |
| Single-stage pilot | 100.798 | Initial check before the measured batch |
| Multi-stage pilot | 73.032 | Initial check before the measured batch |
| Additional multi-stage build | 61.230 | Photographic evidence run |

The three values above are **not** included in `build-times.csv` or in the reported five-run averages.

## Other recorded image measurements

The final image disk usage was 917 MB for the single-stage image and 557 MB for the multi-stage image. Docker's displayed content sizes were 309 MB and 163 MB respectively. These are configuration snapshots, not repeated build-time observations.

See [`../docs/technical-report.pdf`](../docs/technical-report.pdf) for the analysis, limitations and Dockerfile details. The November 2026 application source used for these builds is not bundled in this standalone repository.
