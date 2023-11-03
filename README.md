## Some DASH Additions during execution and development:

* soft links are not appreciated by the input flag (--fastq) cannot detect fastq.gz's etc..
* attempting to pull data using s3+fuse appears to be too slow, rsync data (fastq.gz's) locally or possibly use FSX in production
* using aws cli cp command to pull down locally, faster than rsync :this:
* min vCPU is 4 (t2.medium has capacity 2) - set machine size to at least that!
* do not run as root? no error but only appears to run using sudo under less priv'd account (edited) 
* (no mounting an S3 bucket in local FS for input's)

* working execution:
```sudo nextflow run epi2me-labs/wf-clone-validation --fastq /data/pass --out_dir output/output-poc2DASH```

* where the input (/data/pass) was just copied over locally from S3 to run
```
barcode01  barcode08  barcode15  barcode22  barcode29  barcode36  barcode43  barcode50  barcode57  barcode64  barcode71  barcode78  barcode85  barcode92
barcode02  barcode09  barcode16  barcode23  barcode30  barcode37  barcode44  barcode51  barcode58  barcode65  barcode72  barcode79  barcode86  barcode93
barcode03  barcode10  barcode17  barcode24  barcode31  barcode38  barcode45  barcode52  barcode59  barcode66  barcode73  barcode80  barcode87  barcode94
barcode04  barcode11  barcode18  barcode25  barcode32  barcode39  barcode46  barcode53  barcode60  barcode67  barcode74  barcode81  barcode88  barcode95
barcode05  barcode12  barcode19  barcode26  barcode33  barcode40  barcode47  barcode54  barcode61  barcode68  barcode75  barcode82  barcode89  barcode96
barcode06  barcode13  barcode20  barcode27  barcode34  barcode41  barcode48  barcode55  barcode62  barcode69  barcode76  barcode83  barcode90  unclassified
barcode07  barcode14  barcode21  barcode28  barcode35  barcode42  barcode49  barcode56  barcode63  barcode70  barcode77  barcode84  barcode91
```
* output available at https://aether-bio-data.s3.amazonaws.com/output/output-poc2DASH/wf-clone-validation-report.html
* workflow needs updating

This repository contains a [nextflow](https://www.nextflow.io/) workflow
that can be used to de novo assemble plasmid sequences from Oxford Nanopore data. The workflow allows users to verify complete artificial constructs.

# Clone validation workflow

## Introduction

The workflow accepts FASTQ as the primary input.

Optional inputs:

* A `host_reference` FASTA file.
  
* A primers TSV file if they differ from the default pRham/T7.

The steps of the workflow are as follows - 

* If a `host_reference` is provided, [minimap2](https://github.com/lh3/minimap2) is used to align the reads to the `host_reference`, aligned reads are filtered out.

* The reads are then trimmed at the ends using [Seqkit](https://bioinf.shenwei.me/seqkit/) with the provided `--trim_length` parameter. 

* The sequences are then downsampled using the tool [Rasusa](https://github.com/mbhall88/rasusa).

* [Trycycler](https://github.com/rrwick/Trycycler) is used to create 3 subsamples which are each assembled using [Flye](https://github.com/fenderglass/Flye).

* If there are concatemers in the assembly these are found using minimap2 and deconcatenated using a Python script. 

* Trycycler is used to reconcile the subsampled assemblies into one final assembly. This is polished with [Medaka](https://github.com/nanoporetech/medaka).

* [Seqkit](https://bioinf.shenwei.me/seqkit/) is used to find inserts using the primers supplied to the `--primers` parameter. 

* The assembly is annotated using [pLannotate](https://github.com/barricklab/pLannotate) with the default database containing entries from [fpbase](https://www.fpbase.org/), [Swiss-Prot](https://www.expasy.org/resources/uniprotkb-swiss-prot), [Rfam](https://rfam.org/) and [snapgene](https://www.snapgene.com/). 

* A quality score for the assembly is provided by Medaka.

* Optionally a reference insert sequence can be provided with --insert_reference, which is aligned to the consensus and any variants are reported by [bcftools](https://samtools.github.io/bcftools/bcftools.html).



## Quickstart

The workflow uses [nextflow](https://www.nextflow.io/) to manage compute and 
software resources, as such nextflow will need to be installed before attempting
to run the workflow.

The workflow can currently be run using either
[Docker](https://www.docker.com/products/docker-desktop) or
[singularity](https://docs.sylabs.io/guides/3.0/user-guide/index.html) to provide isolation of
the required software. Both methods are automated out-of-the-box provided
either docker or singularity is installed.

It is not required to clone or download the git repository in order to run the workflow.
For more information on running EPI2ME Labs workflows [visit out website](https://labs.epi2me.io/wfindex).

**Workflow options**

To obtain the workflow, having installed `nextflow`, users can run:

```
nextflow run epi2me-labs/wf-clone-validation --help
```

to see the options for the workflow.

**Workflow outputs**

The primary outputs of the workflow include:

* a consensus fasta file for each sample
* a csv showing the pass or fail status of each sample
* a feature table containing annotations for each of the samples
* an HTML report document detailing the primary findings of the workflow.





## Useful links

* [nextflow](https://www.nextflow.io/)
* [docker](https://www.docker.com/products/docker-desktop)
* [singularity](https://docs.sylabs.io/)
* [minimap2](https://github.com/lh3/minimap2)
* [Seqkit](https://bioinf.shenwei.me/seqkit/)
* [Rasusa](https://github.com/mbhall88/rasusa)
* [Trycycler](https://github.com/rrwick/Trycycler)
* [Flye](https://github.com/fenderglass/Flye)
* [Medaka](https://github.com/nanoporetech/medaka)
* [plannotate tool](https://github.com/barricklab/pLannotate)
* [bcftools](https://samtools.github.io/bcftools/bcftools.html)
