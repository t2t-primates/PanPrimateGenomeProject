# Functional Annotation & Evolutionary Constraint

Integration of PacBio Kinnex full-length transcriptomics and comparative pipelines (Ensembl VEP, OrthoFinder) to map gene family evolution, functional variant constraint, and regulatory sequence conservation.

**Inputs:** Kinnex RNA raw reads (`raw_kinnex_rna`), gene annotations (`annotation_pri_gff3`/`annotation_alt_gff3`), and variant calls (`variants`) from `../../manifests/sample_data_manifest.csv`.

**Outputs:** refined gene models, ortholog groups, variant constraint scores.

```text
annotation/
├── pipeline.nf | Snakefile
├── environment/
├── config/
└── docs.md
```

> Placeholder — pipeline manifest, container spec, and docs to be added.
