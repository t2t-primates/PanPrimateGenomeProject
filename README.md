# Pan-Primate Reference Genome Project (PanPrimate-T2T)

> **⚠️ Placeholder data notice:** This repository is pre-release. NCBI BioProject/BioSample accessions and similar IDs shown throughout are placeholders, not real accessions — they will be replaced once samples are submitted to NCBI. *(Delete this notice once all placeholders are replaced with real values.)*

Of the approximately 500 species in the order Primates, the vast majority still lack high-quality reference genomes-limiting our ability to study primate biology, conservation, evolution, and human health. The **Pan-Primate Reference Genome Project (PanPrimate-T2T)** addresses this gap by generating chromosome-scale, phased *de novo* genome assemblies and annotations for ~40 primate species spanning roughly 70 million years of evolutionary diversification across all major lineages.

All samples are derived from established cell lines maintained by the **Coriell Institute for Medical Research**, ensuring renewable biological resources and long-term reproducibility.

---

## Assemblies & Data Scope

Assemblies are generated using a standardized multi-platform sequencing pipeline:

- **Sequencing:** PacBio HiFi long reads, Oxford Nanopore (ONT) ultra-long reads, chromatin conformation capture (currently Omni-C; future species may use Pore-C or other methods), and PacBio Kinnex full-length transcriptomics.
- **Assembly & Curation:** Assembled with [Verkko](https://github.com/marbl/verkko) and [hifiasm](https://github.com/chhylp123/hifiasm), scaffolded with [YaHS](https://github.com/c-zhou/yahs), and manually curated.

This uniform approach resolves complex genomic regions-including centromeres, segmental duplications, immune loci, and sex chromosomes. Released datasets include haplotype-resolved assemblies, gene annotations, structural and small variant call sets, read alignments, and quality metrics.

The complete collection - spanning raw sequencing signals through whole-genome alignments - is expected to total **~130 TB**.

- **Analysis Pipelines:** See [analyses/README.md](analyses/README.md) for workflows covering whole-genome alignment, structural variation, complex loci, functional annotation, and AI benchmarks. For a step-by-step guide to querying and analyzing the data on AWS, see [`tutorials/getting_started_on_aws.md`](tutorials/getting_started_on_aws.md), or start with the interactive [`tutorials/get-to-know-a-dataset.ipynb`](tutorials/get-to-know-a-dataset.ipynb) notebook.

---

## Species List & Data Access

- **Background & Biological Context:** See [species/](species/README.md) in this repository for photos, names, IUCN status, and assembly status per species.
- **Dataset Manifests:** See [README_dataset.md](README_dataset.md) and [manifests/sample_data_manifest.csv](manifests/sample_data_manifest.csv) for per-sample metadata, accessions, and file locations. For HAL and other multi-species comparative data, see [comparative/README.md](comparative/README.md).

### Cloud & Public Repository Access

- **AWS Open Data:** Hosted for cloud-scale access without downloading multi-terabyte files.
- **NCBI SRA / BioProject:** Archived under BioProject/BioSample accessions listed in `manifests/sample_data_manifest.csv`.

Inspect the S3 bucket hierarchy via AWS CLI:

```bash
aws s3 ls s3://primate-t2t-genomics-open/species_data/ --no-sign-request
```

Full bucket layout, species IDs, and per-sample metadata: [README_dataset.md](README_dataset.md).

---

## Data Reuse and License
All data is released to the public under **Creative Commons Attribution 4.0 International (CC BY 4.0)**.

https://creativecommons.org/licenses/by/4.0/

You are free to share and adapt this material for any purpose, including commercially, provided appropriate credit is given.

---

## Relevant Citations

A DOI and associated publications will be added upon release.

---

## Contact & Team

For general questions regarding the PanPrimate-T2T Project, please contact [psudmant@berkeley.edu](mailto:psudmant@berkeley.edu).

### Project Leadership

* **Peter Sudmant** - [psudmant@berkeley.edu](mailto:psudmant@berkeley.edu)
* **Matthew Mitchell** - [mmitchell@coriell.org](mailto:mmitchell@coriell.org)
* **Erik Garrison** - [egarris5@uthsc.edu](mailto:egarris5@uthsc.edu)
* **Glennis Logsdon** - [glogsdon@pennmedicine.upenn.edu](mailto:glogsdon@pennmedicine.upenn.edu)

#### The Team

- **University of California, Berkeley**
    - Peter Sudmant
    - Scott Ferguson

- **Coriell Institute for Medical Research**
    - Matthew Mitchell
    - Harshleen Chawla

- **University of Tennessee Health Science Center**
    - Erik Garrison
    - Shuo Cao
    - Andrea Guarracino

- **University of Pennsylvania**
    - Glennis Logsdon
    - Shu-Cheng Chuang
    - Keisuke (Keith) Oshima
    - Shenghan Gao
    
- **University of California, Santa Cruz**
    - Prajna Hebbar

---

## History

```
* [date TBD]. v1.0 release - initial release of 10 genomes and associated sequencing, annotation data, and analysis scripts.
```
