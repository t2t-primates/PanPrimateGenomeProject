# Analysis Pipelines & Key Research Outputs

The primary publications accompanying the PanPrimate-T2T project focus on core analytical themes across primate evolution, comparative genomics, and functional biology. Analysis workflows, configuration files, and custom scripts for these studies are housed in dedicated subdirectories within this repository:

* **Whole-Genome Alignment & Synteny ([`alignments/`](alignments/README.md) - includes a runnable pipeline, see below):**
  Multi-species whole-genome alignments using **Progressive Cactus** and **HAL** to investigate large-scale chromosomal rearrangements, synteny conservation, and structural evolution spanning ~70 million years of primate lineage diversification.

* **Structural Variant & Karyotype Evolution ([`structural_variants/`](structural_variants/README.md)):**
  Comprehensive characterization of inversions, translocations, segmental duplications, and complex structural variation using haplotype-resolved T2T assemblies, identifying species-specific vs. evolutionarily conserved structural changes.

* **Complex Loci & Centromeric Regions ([`complex_loci/`](complex_loci/README.md)):**
  Targeted resolution and comparative analysis of previously inaccessible genomic features, including centromeric satellite arrays, major histocompatibility complex (MHC) immune loci, and sex chromosomes across diverse primate clades.

* **Functional Annotation & Evolutionary Constraint ([`annotation/`](annotation/README.md)):**
  Integration of PacBio Kinnex full-length transcriptomics and comparative pipelines (Ensembl VEP, OrthoFinder) to map gene family evolution, functional variant constraint, and regulatory sequence conservation.

* **Genomic AI & Benchmark Models ([`benchmarks/`](benchmarks/README.md)):**
  Standardized benchmark sets and data preparation scripts for training DNA foundation models, variant effect predictors, and basecalling/methylation detection algorithms.

> **Note on Reproducibility:** `alignments/` currently contains a runnable Snakemake pipeline, container spec, and config - written and manually reviewed, not yet executed against real data (see its README for details). The other four subdirectories are placeholders; each will follow the same pattern (pipeline manifest, containerized environment spec, and step-by-step documentation) as they're built out.

Each subdirectory's inputs are the released data products described in [`../README_dataset.md`](../README_dataset.md) - assemblies, annotations, variant calls, and comparative alignments, referenced by S3 path or `genome_id` from [`../manifests/sample_data_manifest.csv`](../manifests/sample_data_manifest.csv).
