# Comparative and multi-species data

Whole-genome alignments that span **multiple species** — HAL and comparative MAF — don't fit the per-accession model documented in [`../README_dataset.md`](../README_dataset.md). This directory documents them separately for that reason:

| | `species_data/<accession_id>/...` | `comparative/...` |
| --- | --- | --- |
| Scope | One sample | Many species at once |
| "Version" means | One assembly's revision (`genome_version`) | Which *set* of assembly versions across species went into a build |
| Changes when | That sample is re-assembled/re-curated | Any included species' assembly changes, or a species is added/removed |

## Layout

```text
s3://primate-t2t-genomics-open/
└── comparative/
    ├── comparative_manifest.csv     # one row per HAL build — which genome_ids it includes
    ├── hal/
    │   └── all_species_build<N>.hal
    └── maf/
        └── <genome_id>_vs_human.maf
```

## MAF (pairwise, one species vs. human)

`comparative/maf/<genome_id>_vs_human.maf` — named with the full `genome_id` (e.g. `PR00232_2.0_vs_human.maf`), same self-describing convention as version-specific files in `species_data/`. A MAF is regenerated whenever the non-human side's assembly changes, so each `genome_id` a species has ever had can have its own MAF. This is what `sample_data_manifest.csv`'s `alignment_comparative` column points to — one path, specific to that row's exact assembly version.

## HAL (multi-species graph alignment)

`comparative/hal/all_species_build<N>.hal` — the full multi-species alignment graph. Unlike MAF, this isn't naturally tied to one `genome_id`, since it's built from many species at once. `build<N>` is a simple incrementing counter, not a `major.minor` pair — a new build is cut whenever the set of included assemblies changes meaningfully enough to warrant a new graph (a new species added, or enough re-assemblies accumulated). It is **not** regenerated for every single per-species version bump.

## `comparative_manifest.csv`

Since a HAL build spans many species and versions, one HAL path alone doesn't say what went into it — `comparative_manifest.csv` records that explicitly, one row per build:

| Column | Description |
| --- | --- |
| `build_id` | Primary key, e.g. `build3` |
| `hal_path` | S3 path to the HAL file for this build |
| `included_genome_ids` | Colon-delimited (`:`) list of every `genome_id` included in this build, e.g. `PR00232_2.0:PR00036_1.0:...` |
| `release_date` | When this build was published |
| `notes` | Free text — e.g. "added Papio anubis" |

To find which exact assembly version of a species was used in a given HAL build, cross-reference `included_genome_ids` against `sample_data_manifest.csv`'s `genome_id` column — this is the same join pattern used elsewhere in the dataset (one manifest per data category, joined on a shared ID).

## Example

```csv
build_id,hal_path,included_genome_ids,release_date,notes
build1,comparative/hal/all_species_build1.hal,PR00232_1.0,2026-01-15,Initial build — Drill only
build2,comparative/hal/all_species_build2.hal,PR00232_1.1,2026-02-01,Drill re-curated (misjoin fix)
build3,comparative/hal/all_species_build3.hal,PR00232_2.0:PR00036_1.0,2026-04-10,Drill v2.0 (new ONT data) + Olive Baboon added
```
