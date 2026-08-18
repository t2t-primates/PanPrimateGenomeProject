# PanPrimate-T2T dataset

> **⚠️ Placeholder data notice:** This repository is pre-release. NCBI BioProject/BioSample accessions and similar IDs shown throughout (including in the manifest and JSON examples below) are placeholders, not real accessions — they will be replaced once samples are submitted to NCBI. *(Delete this notice once all placeholders are replaced with real values.)*

## Species ID

| species_id | genus | species | common_name | accession_id | assembly_status |
| --- | --- | --- | --- | --- | --- |
| mandrillus_leucophaeus | Mandrillus | leucophaeus | Drill | PR00232 | released |
| papio_anubis | Papio | anubis | Olive Baboon | PR00036 | in_progress |
| eulemur_rubriventer | Eulemur | rubriventer | Red-bellied Lemur | PR00129 | planned |

`accession_id` is the identifier used throughout the S3 bucket path (`species_data/<accession_id>/...`) and in raw/version-specific filenames; `species_id` is a taxonomic identifier, not a path component. This table is illustrative - the authoritative, current record for every sample (not just species) is `manifests/sample_data_manifest.csv` (see below). For the full, current species list across all phases, see [`../species/README.md`](species/README.md).

## Sample data manifest

`manifests/sample_data_manifest.csv` is the single metadata table for the whole dataset - one row per released assembly version, with every accession and every data file location as a column. A sample that has been re-assembled or re-curated has one row per version it has ever shipped (eg. `PR00232_1.0` and `PR00232_1.1` are two separate rows), so no history is ever overwritten and every `genome_id` that was ever cited stays resolvable. Sample-identity fields (species, sex, accessions) are repeated identically across that sample's version rows. Raw data columns (`raw_ont`, `raw_pacbio_hifi`, `raw_kinnex_rna`) hold a colon-delimited (`:`) list of the run file(s) that fed that version's assembly - usually the same list across a sample's versions, but not always: raw sequencing is stored as one file per run (never merged), and if a new assembly version incorporates an additional run for a platform (eg. a second ONT run added for deeper coverage), that row's column gains the new run's path while keeping the earlier one, and the earlier version's row is left exactly as it was. `raw_chromatin_capture` follows the same colon-delimited, append-only, per-run pattern, but holds SRA run accessions rather than S3 paths - see [Chromatin Capture](#chromatin-capture-omni-c--pore-c) below. A published row is never rewritten after the fact, and no raw file is ever overwritten in place; new raw data always arrives as a new, separate run file (we use "append-only" to refer to this data method).

| Index | Column | Description |
| --- | --- | --- |
| 1 | `genome_id` | Primary key. `<accession_id>_<genome_version>`, eg. `PR00232_1.1`. One row per `genome_id`. Cite this when referencing a specific assembly. Blank for samples with no released version yet. |
| 2 | `accession_id` | Coriell catalog ID for the sample. Repeats across all of that sample's version rows. |
| 3 | `genome_version` | Version of *this specific row's* assembly, eg. `1.0`, `1.1` - matches the `v<major>.<minor>/` directory in S3. |
| 4 | `is_latest` | `TRUE` for the one row per sample that is the current/most-recent released version, `FALSE` for all superseded versions. Filter on this to get "current state of the dataset." |
| 5 | `release_date` | Date this specific version was published, `YYYY-MM-DD`. Blank for samples with no released version yet. |
| 6 | `species_id` | Taxonomic identifier, derived as lowercase `<genus>_<species>` - eg. `genus=Mandrillus`, `species=leucophaeus` → `species_id=mandrillus_leucophaeus`. Descriptive only - `accession_id`, not `species_id`, is the bucket path component. |
| 7 | `genus` | Genus, capitalized (eg. `Mandrillus`) |
| 8 | `species` | Species epithet, lowercase (eg. `leucophaeus`) |
| 9 | `common_name` | Common species name |
| 10 | `sex` | `male` / `female` / `unknown` |
| 11 | `biosample` | NCBI BioSample accession |
| 12 | `bioproject` | NCBI BioProject accession |
| 13 | `assembly_status` | `planned` / `in_progress` / `released` |
| 14 | `genome_hap1` | NCBI GenBank Assembly accession for the haplotype 1 assembly (e.g. `GCA_900000001.1`) — hosted at NCBI, not AWS. View at `https://www.ncbi.nlm.nih.gov/datasets/genome/<accession>/` |
| 15 | `genome_hap2` | NCBI GenBank Assembly accession for the haplotype 2 assembly (e.g. `GCA_900000002.1`) — hosted at NCBI, not AWS. View at `https://www.ncbi.nlm.nih.gov/datasets/genome/<accession>/` |
| 16 | `genome_pri` | S3 path to the primary assembly (longer sequence per chromosome pair; see [Primary + alternate assembly](#bucket-layout-and-download) below) |
| 17 | `genome_alt` | S3 path to the alternate assembly (shorter sequence per chromosome pair) |
| 18 | `annotation_pri_gff3` | S3 path to gene annotation, called on the primary assembly |
| 19 | `annotation_alt_gff3` | S3 path to gene annotation, lifted over onto the alternate assembly |
| 20 | `repeat_masker_pri` | S3 path to RepeatMasker output, called on the primary assembly |
| 21 | `repeat_masker_alt` | S3 path to RepeatMasker output, called on the alternate assembly |
| 22 | `repeat_families` | S3 path to the RepeatModeler-generated repeat family consensus library (FASTA) for this assembly — used as RepeatMasker's custom library for annotation, and also useful on its own for directly examining this species' repeat families. One consensus sequence per family; headers follow RepeatModeler's `family_name#class/family` convention (e.g. `ltr-1_family-1#LTR/ERVK`) |
| 23 | `methylation` | S3 path to the ONT-based methylation output **directory** for this version (5mC/5hmC calls) - see [Methylation](#methylation) below |
| 24 | `fiberseq` | S3 path to the Fiber-seq output **directory** for this version (m6A, MSPs, nucleosome calls) - see [Fiber-seq](#fiber-seq) below |
| 25 | `variants` | S3 path to the variant calls **directory** for this version (see the file-naming table below for what's inside) |
| 26 | `raw_ont` | Colon-delimited list of S3 paths to ONT run file(s) used for this version |
| 27 | `raw_pacbio_hifi` | Colon-delimited list of S3 paths to PacBio HiFi run file(s) used for this version |
| 28 | `raw_chromatin_capture` | Colon-delimited list of SRA run accessions for chromatin capture run(s) used for this version — reads are archived at NCBI/SRA (FASTQ), not hosted on AWS. See [Chromatin Capture](#chromatin-capture-omni-c--pore-c) below |
| 29 | `raw_kinnex_rna` | Colon-delimited list of S3 paths to Kinnex RNA run file(s) used for this version |
| 30 | `alignment_reads` | S3 path to HiFi reads aligned to the assembly |
| 31 | `alignment_comparative` | S3 path to the comparative (MAF) alignment - see [Comparative and multi-species data](#comparative-and-multi-species-data) below |

Samples with no released version yet (`assembly_status` = `planned` or `in_progress`) have a single row with `genome_id`, `genome_version`, `is_latest`, and all data-path columns blank.

**Versioning is per-sample, per-assembly, and lives directly in the S3 path** - there is no single bucket-wide release version. `raw_sequencing/` is append-only: new files can be added when more sequencing is generated for a sample, but existing files are never modified or removed, so any file a past version's manifest row points to remains exactly as it was. Each re-assembly or re-curation gets its own `v<major>.<minor>/` directory containing that version's `assembly/`, `annotation/`, `variants/`, and `alignments/`. See [Bucket layout](#bucket-layout-and-download) below. `genome_version` in the manifest always matches the version directory that row's paths point into, and `genome_id` (`<accession_id>_<genome_version>`) is what you cite for a specific assembly.

**`genome_version` follows `<major>.<minor>`, and the two numbers mean different things:**

| Bump | Meaning | Example |
| --- | --- | --- |
| **Minor** (`1.0` → `1.1`) | Error fix or re-scaffolding on the *same* underlying raw data - no new sequencing incorporated | Fixing a misjoin, improving scaffolding with the existing Omni-C/Pore-C data |
| **Major** (`1.1` → `2.0`) | Full re-assembly, which *may* include newly incorporated raw data | Adding a second ONT run for deeper coverage and re-assembling from scratch |

A minor bump's `raw_*` columns are always identical to the version it patches. A major bump's `raw_*` columns may point at newer, additional raw files if new data was incorporated (see below) - or may stay the same if the re-assembly used identical inputs with a different pipeline/parameters.

Every version a sample has ever had ships as its own row - nothing is overwritten when a new version is released, so a previously-cited `genome_id` always stays resolvable in the manifest. Filter on `is_latest == TRUE` to get one row per sample reflecting only the current state of the dataset.

```bash
# Pull the manifest and list every currently-latest released genome's ID and S3 path (primary assembly)
aws s3 cp --no-sign-request \
  s3://primate-t2t-genomics-open/manifests/sample_data_manifest.csv - \
  | awk -F, '$13 == "released" && $4 == "TRUE" { print $1, $16 }'

# List every version that has ever existed for one sample
aws s3 cp --no-sign-request \
  s3://primate-t2t-genomics-open/manifests/sample_data_manifest.csv - \
  | awk -F, '$2 == "PR00232" { print $1, $3, $4 }'
```

## AWS Access & Compute

For a complete, step-by-step walkthrough (listing data, streaming regions, running compute), see [`tutorials/getting_started_on_aws.md`](tutorials/getting_started_on_aws.md) or the interactive [`tutorials/get-to-know-a-dataset.ipynb`](tutorials/get-to-know-a-dataset.ipynb) notebook. The examples below are quick reference.

All indexed files (BAM `.bai`, bgzip VCF/GFF3/BED `.tbi`, bgzip FASTA `.fai`) support HTTP byte-range requests directly from S3 - you can pull a single genomic interval without downloading the file, using standard tools:

```bash
# A region of aligned reads
samtools view \
  https://primate-t2t-genomics-open.s3.amazonaws.com/species_data/PR00232/v1.0/alignments/PR00232_1.0.hifi.sorted.bam \
  chr7:1000000-1050000

# The same region's variant calls
bcftools view \
  https://primate-t2t-genomics-open.s3.amazonaws.com/species_data/PR00232/v1.0/variants/PR00232_1.0.vcf.gz \
  chr7:1000000-1050000

# The same region's gene annotation (primary assembly)
tabix \
  https://primate-t2t-genomics-open.s3.amazonaws.com/species_data/PR00232/v1.0/annotation/PR00232_1.0.pri.gff3.gz \
  chr7:1000000-1050000
```

IGV and JBrowse can also load these S3 URLs directly as remote tracks, without downloading anything locally.

**Running compute directly against S3**, rather than copying multi-terabyte files to local or attached storage:

| Service | Use |
| --- | --- |
| **Amazon EC2** | Mount via `mountpoint-s3`/`s3fs`, or point `samtools`/`bcftools` directly at S3 URLs |
| **AWS Batch** | Containerized jobs reading/writing `s3://primate-t2t-genomics-open/...` directly, eg. array jobs iterating over `sample_data_manifest.csv` |
| **Amazon SageMaker** | Training/processing jobs with the bucket as an S3 input channel - eg. combining raw ONT/HiFi signal with assemblies for genomic foundation model training |
| **AWS HealthOmics** | Whole-genome alignment, variant discovery, and comparative genomics workflows using the released BAM/VCF/assembly files as inputs |

## Bucket layout and download

Data lives at:

```text
s3://primate-t2t-genomics-open/
```

No AWS credentials are required (`--no-sign-request`).

```bash
# Everything for one sample (all versions + raw data)
aws s3 cp --no-sign-request --recursive \
  s3://primate-t2t-genomics-open/species_data/PR00232/ .

# Just the current released assembly (v1.0)
aws s3 cp --no-sign-request --recursive \
  s3://primate-t2t-genomics-open/species_data/PR00232/v1.0/ .
```

Layout:

```text
s3://primate-t2t-genomics-open/
├── manifests/
│   └── sample_data_manifest.csv        # one row per sample - accessions + current-version paths
│
└── species_data/
    └── <accession_id>/
        ├── raw_sequencing/              # APPEND-ONLY - one file per sequencing run, never merged or overwritten
        │   ├── ont/                     # POD5 raw signal + FASTQ.GZ per run
        │   ├── pacbio_hifi/             # Unaligned HiFi BAM + FASTQ.GZ per run
        │   └── kinnex_rna/              # Isoform RNA BAM + FASTQ.GZ per run
        │
        ├── metadata/                    # APPEND-ONLY - sample-level, not tied to any assembly version; new runs get appended, nothing is edited/removed
        │   └── <accession_id>_sample_metadata.json
        │
        ├── v1.0/                        # Assembly release 1.0
        │   ├── assembly/                # primary/alternate FASTA files, QC
        │   ├── annotation/              # GFF3 gene models (pri + alt), RepeatMasker + repeat family output
        │   ├── variants/                # VCF files
        │   ├── alignments/              # Reads aligned to the v1.0 assembly
        │   ├── methylation/             # methylation calls
        │   │   └── ont/
        │   └── fiberseq/                # PacBio Fiber-seq output
        │
        └── v1.1/                        # Assembly release 1.1 (patched/re-curated)
            ├── assembly/
            ├── annotation/
            ├── variants/
            ├── alignments/
            ├── methylation/
            │   └── ont/
            └── fiberseq/
```

Raw sequencing is generated once per sample and never re-derived per assembly version, so it lives outside the `v*/` directories. Everything version-specific - the assembly itself and everything built from it (annotation, variant calls, read alignments, methylation, Fiber-seq) - is fully contained within its own `v<major>.<minor>/` directory, so two versions never overwrite or depend on each other.

**Version-specific files are named with the full `genome_id`, not just `accession_id`:**

| File | Pattern | Example |
| --- | --- | --- |
| Primary assembly | `<genome_id>.pri.fa.gz` | `PR00232_1.1.pri.fa.gz` |
| Alternate assembly | `<genome_id>.alt.fa.gz` | `PR00232_1.1.alt.fa.gz` |
| Gene annotation (primary) | `<genome_id>.pri.gff3.gz` | `PR00232_1.1.pri.gff3.gz` |
| Gene annotation (alternate, lifted over) | `<genome_id>.alt.gff3.gz` | `PR00232_1.1.alt.gff3.gz` |
| Repeat family consensus library | `<genome_id>.repeat_families.fasta.gz` | `PR00232_1.1.repeat_families.fasta.gz` |
| Variant calls | `<genome_id>.vcf.gz` | `PR00232_1.1.vcf.gz` |
| Read alignment | `<genome_id>.hifi.sorted.bam` | `PR00232_1.1.hifi.sorted.bam` |

**Haplotype assemblies (`hap1`/`hap2`) are archived at NCBI, not hosted on AWS.** Following the same `<genome_id>.hap1.fa.gz` / `<genome_id>.hap2.fa.gz` naming pattern, they're submitted to NCBI as part of each release rather than duplicated in this bucket — see [NCBI cross-reference](#ncbi-cross-reference) below for how to locate them via the manifest's `biosample`/`bioproject` columns.

**Primary + alternate assembly:** For annotation purposes and analyses that require only a single haplotype per genome, we also generate a primary (`pri`) and alternate (`alt`) assembly. For each autosome pair, the longer of the two haplotype sequences becomes the primary chromosome, and the shorter becomes the alternate. eg. if `chr1_hap1` is longer than `chr1_hap2`, then `chr1_pri` = `chr1_hap1` and `chr1_alt` = `chr1_hap2`. Sex chromosomes (X and/or Y, whichever are present in the sequenced individual) are included in the primary assembly only. Unlike `hap1`/`hap2`, the primary/alternate assemblies are hosted here on AWS.

Unplaced/unscaffolded contigs are not included in the primary or alternate assembly - they remain only in the haplotype assemblies (`hap1`/`hap2`). Each haplotype includes its own complete set of unscaffolded contigs; unphased contigs (those that couldn't be assigned to either haplotype) are included in `hap1` only, not `hap2`.

Annotation is called on the primary assembly and lifted over onto the alternate assembly - `annotation_pri_gff3` is the primary gene-model call, `annotation_alt_gff3` is derived from it via liftover, not called independently. Neither `hap1` nor `hap2` receives a standalone annotation; instead, their annotations are extracted from the primary and alternate sets.

Note: `hap1`/`hap2` labels follow the assembler's own haplotype assignment (Verkko/hifiasm) and carry no inherent meaning - `hap1` is not consistently the larger, primary-leaning, or otherwise "preferred" haplotype across samples. `pri`/`alt` is the only pairing where "which one is longer" is meaningful per chromosome; treat `hap1`/`hap2` purely as an arbitrary assembler label.

**Raw file storage:** each sequencing run is stored as its own file, named `<accession_id>_<platform>_run<N>.<ext>` (eg. `PR00232_hifi_run1.bam`) inside that platform's directory - runs are **never merged** into a single combined file. When new sequencing arrives for a platform, it's added as a new `run<N>` file alongside the existing ones; nothing is ever overwritten or replaced. A sample can have multiple run files per platform sitting in the same directory. The manifest's `raw_*` columns record exactly which run file(s) fed a given assembly version as a colon-delimited list (see above) - this is how "which combination of files" is answered without needing a separate lookup.

**Chromatin capture is archived at NCBI/SRA, not hosted on AWS:** unlike HiFi, ONT, and Kinnex, chromatin capture (Hi-C/Omni-C/Pore-C) reads are not part of this bucket - they're submitted to SRA as paired-end FASTQ (R1/R2), consistent with this being short-read sequencing rather than the single-molecule long-read platforms hosted here. The `raw_chromatin_capture` manifest column holds the SRA run accession for each run (colon-delimited if multiple), not an S3 path.

**Every run also ships a bgzip-compressed FASTQ alongside its native file** - `<accession_id>_<platform>_run<N>.fastq.gz` sits next to `..._run<N>.bam` (or `.pod5`) in the same directory, sharing the same `run_id`. This isn't a separate manifest entry: since it's always the same basename with a different extension, it's fully implied by the `raw_*` path already in the manifest - swap the extension to get it. FASTQ files are provided, especially for ONT, for tools that can only make use of this data type or to allow direct streaming of sequence data.

**Why not merge runs into one file per platform?** Merging (eg. `pod5 merge`) would mean re-storing every prior run's data again each time new data arrives - at this dataset's scale, that duplicates multi-TB files repeatedly for no benefit. Assembly and basecalling tools (hifiasm, verkko, dorado) all accept multiple input files natively, so there's no processing reason to pre-merge; a user who wants one combined file can merge on demand from the exact run files listed in the manifest.

**How this interacts with region-based streaming:** the byte-range/streaming examples in [AWS Access & Compute](#aws-access--compute) apply to **indexed, per-version outputs** (`alignments/`, `variants/`, `annotation/`) - files with a `.bai`/`.tbi`/`.fai` index built for genomic-coordinate lookup. Raw sequencing was never part of that pattern: it isn't aligned to anything yet, so there's no coordinate to seek to. Consuming raw data means fetching the specific run file(s) you need (from the manifest's colon-delimited list) and feeding them to a processing tool as a file list - not a byte-range query. These are two different access patterns for two different kinds of file, not a limitation introduced by storing multiple runs per platform.

## PacBio HiFi

`raw_sequencing/pacbio_hifi/<accession_id>_hifi_run<N>.bam` (+ `..._run<N>.fastq.gz`) - unaligned HiFi reads, one file per run (never merged). The primary assembly input for Verkko/hifiasm, which accept multiple run files directly. Coordinate-sorted alignment against a specific assembly version is at `species_data/<accession_id>/<version>/alignments/<genome_id>.hifi.sorted.bam` (+ `.bai`).

## ONT

`raw_sequencing/ont/<accession_id>_ont_run<N>.pod5` (+ `..._run<N>.fastq.gz` **derived** basecalled) - raw signal per run, not just basecalled reads, so the data supports basecaller/methylation-model benchmarking directly. Basecalling tools (eg. `dorado`) accept a directory or list of POD5 files natively; the FASTQ is provided as a convenience for tools that need basecalled reads directly rather than raw signal. Basecalling model and version used for the shipped calls, if any accompany the POD5, are noted per-run in `metadata/<accession_id>_sample_metadata.json` (`sequencing.ont[N].chemistry`, matched by `run_id`).

## Chromatin Capture (Omni-C / Pore-C)

Chromatin capture (Hi-C/Omni-C/Pore-C) reads are used for YaHS scaffolding to chromosome scale, but - unlike every other raw sequencing type in this dataset - **the reads themselves are not hosted on AWS**. They're archived at NCBI/SRA as paired-end FASTQ (R1/R2), consistent with publication requirements, rather than duplicated in this bucket. The `raw_chromatin_capture` manifest column holds the SRA run accession (colon-delimited if a version incorporated multiple runs) rather than an S3 path. The actual method and kit/protocol version per run are recorded in metadata (`sequencing.chromatin_capture[N].method`, `sequencing.chromatin_capture[N].kit_version`) - the directory and manifest column are method-agnostic since the project currently uses **Omni-C** but may use **Pore-C** or other chromatin capture methods for future species.

## Kinnex RNA

`raw_sequencing/kinnex_rna/<accession_id>_kinnex_run<N>.bam` (+ `..._run<N>.fastq.gz`) - PacBio Kinnex full-length transcriptome reads, one file per run, used to annotate genomes for a given assembly version.

## Methylation

`<version>/methylation/ont/` - 5mC/5hmC methylation calls from ONT basecalling (eg. via `dorado` modbase calling + `modkit`), derived from the same raw ONT signal used for assembly. Points to a directory since this may include multiple representations (per-read calls, aggregated bedMethyl, etc.) rather than a single file. `methylation` in the manifest points here.

## Fiber-seq

`<version>/fiberseq/` - Fiber-seq output (via `fibertools-rs`): m6A methylation, CpG methylation, MSPs (accessible regions), and nucleosome calls. This is inherently multi-track and doesn't collapse into one file, so `fiberseq` in the manifest points to this directory rather than a specific file.

## Sample metadata

Every sample has a full metadata record at `metadata/<accession_id>_sample_metadata.json`. Real-format example: [`examples/PR00232_sample_metadata.json`](examples/PR00232_sample_metadata.json).

```json
{
  "species_id": "mandrillus_leucophaeus",
  "accession_id": "PR00232",
  "sex": "male",
  "sequencing": {
    "ont": [
      { "run_id": "run1", "instrument": "PromethION 24", "chemistry": "R10.4.1", "run_accession": "SRR00000010" },
      { "run_id": "run2", "instrument": "PromethION 2 Solo", "chemistry": "R10.4.1", "run_accession": "SRR00000014" }
    ],
    "pacbio_hifi": [
      { "run_id": "run1", "instrument": "Revio", "chemistry": "SMRTbell prep kit 3.0", "run_accession": "SRR00000011" }
    ],
    "chromatin_capture": [
      { "run_id": "run1", "method": "omni_c", "kit_version": "Dovetail Omni-C v2", "run_accession": "SRR00000012" }
    ],
    "kinnex_rna": [
      { "run_id": "run1", "instrument": "Revio", "run_accession": "SRR00000013" }
    ]
  },
  "ncbi_biosample": "SAMN00000001"
}
```

Each `run_id` (`run1`, `run2`, ...) matches the `_run<N>` suffix in that platform's raw filenames, so a manifest path like `..._ont_run2.pod5` can be looked up directly against `sequencing.ont[1]` in this file for its instrument/chemistry/accession. New runs are appended to the relevant platform's list as they're generated - existing entries are never edited or removed.

The per-sample JSON carries richer detail (instrument, chemistry) than fits in a flat CSV column; `sample_data_manifest.csv` is the fast index, this JSON is the full record - use the manifest to find the sample, then fetch its JSON for full detail if needed.

## NCBI cross-reference

Every sample's NCBI accessions are columns in `sample_data_manifest.csv` (`bioproject`, `biosample`). Raw reads and finished assemblies are additionally archived at NCBI; AWS hosts the full cloud-optimized lineage (raw signal through comparative alignments) for interactive, region-based analysis.

## Comparative and multi-species data

HAL alignments and other data products that span multiple species (rather than belonging to one sample) don't fit the per-accession model above. See [`comparative/README.md`](comparative/README.md) for how that data is organized, versioned, and named.
