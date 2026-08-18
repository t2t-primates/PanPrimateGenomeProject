# Whole-Genome Alignment & Synteny

Multi-species whole-genome alignments using **Progressive Cactus** and **HAL**, investigating large-scale chromosomal rearrangements, synteny conservation, and structural evolution spanning ~70 million years of primate lineage diversification.

**Inputs:** primary (`pri`) assembly (`genome_pri` in `../../manifests/sample_data_manifest.csv`) for every currently-released species.

**Outputs:** pairwise (MAF) and multi-species (HAL) alignments, published under the shared top-level `comparative/` prefix - see [`../../comparative/README.md`](../../comparative/README.md) for the full layout, naming convention, and `comparative_manifest.csv` (which genome_ids went into each HAL build), since these outputs span all species and aren't tied to any single sample's assembly version.

## Running this pipeline

```text
alignments/
├── Snakefile                      # Progressive Cactus / HAL workflow definition
├── environment/                   # Container spec
└── config/
    ├── config.yaml                # Bucket, manifest path, build ID, container image
    └── species_tree.nwk           # Species tree covering all included accession_ids
```

```bash
# Smoke-test the pipeline structure without running real alignment (recommended first step)
snakemake -n --configfile config/config.yaml

# Run for real, against released data on S3
snakemake --configfile config/config.yaml --cores 32 --use-singularity
```

The pipeline: pulls each released genome's primary assembly from S3, builds a Cactus seqFile from the species tree in `config/species_tree.nwk`, runs Progressive Cactus to produce the multi-species HAL, extracts pairwise MAFs (each species vs. human), and emits a manifest row matching `comparative_manifest.csv`'s schema for review before being committed.

**Known limitations, honestly stated:**
- `config/species_tree.nwk` currently ships with a small **illustrative placeholder tree** (3 species), not the real topology for all ~40 species - replace with a real reference tree (e.g. from [TimeTree.org](http://timetree.org)) covering every `accession_id` before running a production build.
- This pipeline was written and manually reviewed against the real manifest schema, but **has not been run against real genome assemblies**, since no assemblies are public yet. Treat this as a plausible pipeline, not one with a verified test run. Run `snakemake -n --configfile config/config.yaml` yourself before trusting it further.
- `cactus_align`'s job-store naming (`jobstore_{BUILD_ID}`) is a placeholder pattern - Cactus job stores need a genuinely unique, persistent path per run in production use; adjust before scaling to concurrent builds.
