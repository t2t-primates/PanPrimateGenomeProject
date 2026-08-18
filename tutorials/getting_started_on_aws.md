# Getting Started: Querying and Analyzing PanPrimate-T2T Data on AWS

A complete, runnable walkthrough - from "I've never touched this bucket" to streaming a real genomic region and running compute against it, using the currently-released Drill (*Mandrillus leucophaeus*, `PR00232`) genome as the worked example throughout.

**Prerequisites:** AWS CLI v2, `samtools`/`bcftools`/`tabix` (via `conda install -c bioconda samtools bcftools htslib`), Python 3. No AWS account or credentials are required for any step below - the bucket is public read (`--no-sign-request`).

---

## Step 1 - List what's available

Every species and every released version is indexed in one manifest file. Start there rather than browsing the bucket blindly:

```bash
aws s3 cp --no-sign-request \
  s3://primate-t2t-genomics-open/manifests/sample_data_manifest.csv - \
  | head -1
```

Find every currently-latest released genome and its primary assembly path:

```bash
aws s3 cp --no-sign-request \
  s3://primate-t2t-genomics-open/manifests/sample_data_manifest.csv - \
  | awk -F, '$13 == "released" && $4 == "TRUE" { print $1, $16 }'
```

This should print `PR00232_2.0` and its `genome_pri` S3 path - that's the genome we'll use for the rest of this tutorial.

## Step 2 - See everything available for one sample

```bash
aws s3 ls --no-sign-request --recursive \
  s3://primate-t2t-genomics-open/species_data/PR00232/ \
  | head -20
```

You'll see `raw_sequencing/` (append-only, shared across versions), `metadata/`, and one directory per assembly version (`v1.0/`, `v1.1/`, `v2.0/`).

## Step 3 - Stream a genomic region without downloading anything

This is the core cloud-native pattern this dataset is built around: every indexed file supports HTTP byte-range requests, so you pull exactly the interval you need.

```bash
# A region of aligned HiFi reads
samtools view \
  https://primate-t2t-genomics-open.s3.amazonaws.com/species_data/PR00232/v2.0/alignments/PR00232_2.0.hifi.sorted.bam \
  chr7:1000000-1050000

# The same region's variant calls
bcftools view \
  https://primate-t2t-genomics-open.s3.amazonaws.com/species_data/PR00232/v2.0/variants/PR00232_2.0.vcf.gz \
  chr7:1000000-1050000

# The same region's gene annotation
tabix \
  https://primate-t2t-genomics-open.s3.amazonaws.com/species_data/PR00232/v2.0/annotation/PR00232_2.0.pri.gff3.gz \
  chr7:1000000-1050000
```

Each command downloaded only the bytes for that interval - not the full multi-gigabyte file. You can point IGV or JBrowse at these same URLs directly as remote tracks for interactive browsing instead of the command line.

## Step 4 - Pull a sample's full metadata record

```bash
aws s3 cp --no-sign-request \
  s3://primate-t2t-genomics-open/species_data/PR00232/metadata/PR00232_sample_metadata.json - \
  | python3 -m json.tool
```

This gives you per-run sequencing detail (instrument, chemistry, run accession) that doesn't fit in the flat manifest CSV.

## Step 5 - Run compute directly against S3 (no local copy)

**Local/EC2, ad hoc:** everything above already ran without downloading full files - the same commands work identically on an EC2 instance, just faster (same-region network).

**AWS Batch, at scale:** iterate the manifest to process every released sample as an array job, without ever pulling files to local disk:

```bash
# Generate one job per released sample from the manifest
aws s3 cp --no-sign-request \
  s3://primate-t2t-genomics-open/manifests/sample_data_manifest.csv - \
  | awk -F, '$13 == "released" && $4 == "TRUE" { print $1 }' \
  > released_genome_ids.txt

# Submit one Batch job per line, each job reading directly from S3
while read genome_id; do
  aws batch submit-job \
    --job-name "analyze-${genome_id}" \
    --job-queue your-queue \
    --job-definition your-job-definition \
    --parameters genome_id="${genome_id}"
done < released_genome_ids.txt
```

Each job's container just needs `samtools`/`bcftools` (or your own tool) pointed at the S3 URLs from Steps 1–3 - no data staging step required.

## Where to go next

- [`../README_dataset.md`](../README_dataset.md) - full bucket layout, file naming, and manifest schema reference
- [`../comparative/README.md`](../comparative/README.md) - multi-species HAL/MAF alignments (a different access pattern than per-sample data)
- [`../analyses/`](../analyses/) - the actual pipelines used to produce this data, if you want to reproduce or extend them
- [`../species/README.md`](../species/README.md) - browse by species for photos, IUCN status, and per-species assembly status

**Note on current data state:** as of this writing, `PR00232` (Drill) is the only genome with real, complete data - used throughout this tutorial for that reason. All commands above are written against real, currently-valid S3 paths for that sample. Other species will follow the identical access pattern once released; swap in their `accession_id` and current `genome_version` from the manifest.
