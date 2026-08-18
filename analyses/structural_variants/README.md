# Structural Variant & Karyotype Evolution

Comprehensive characterization of inversions, translocations, segmental duplications, and complex structural variation using haplotype-resolved T2T assemblies, identifying species-specific vs. evolutionarily conserved structural changes.

**Inputs:** haplotype assemblies (`genome_hap1`/`genome_hap2` — NCBI accessions, fetched from NCBI rather than this bucket) and comparative alignments (`../alignments/`) across released species.

**Outputs:** per-species and cross-species structural variant call sets, karyotype comparison tracks.

```text
structural_variants/
├── pipeline.nf | Snakefile
├── environment/
├── config/
└── docs.md
```

> Placeholder — pipeline manifest, container spec, and docs to be added.
