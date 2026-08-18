# Genomic AI & Benchmark Models

Standardized benchmark sets and data preparation scripts for training DNA foundation models, variant effect predictors, and basecalling/methylation detection algorithms.

**Inputs:** raw ONT signal (`raw_ont`), PacBio HiFi reads (`raw_pacbio_hifi`), assemblies, annotations, and variant calls — the full data lineage referenced in `../../manifests/sample_data_manifest.csv`.

**Outputs:** curated train/validation/test splits, benchmark task definitions, baseline model results.

```text
benchmarks/
├── pipeline.nf | Snakefile
├── environment/
├── config/
└── docs.md
```

> Placeholder — pipeline manifest, container spec, and docs to be added.
