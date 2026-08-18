# Environment

This pipeline runs inside the official Cactus/HAL toolkit container, pinned by version for reproducibility:

```
docker://quay.io/comparative-genomics-toolkit/cactus:v2.9.3
```

Referenced by `cactus_container` in `../config/config.yaml`, and used automatically by Snakemake's `--use-singularity` / `--use-apptainer` flag (or Docker equivalent) — no separate local install of Cactus/HAL is required.

Additional requirements, installed on the host (not inside the container):
- `snakemake` >= 8.0
- `awscli` >= 2.0 (for `aws s3 cp` staging steps)
- Python >= 3.10 (Snakemake's own dependency)
