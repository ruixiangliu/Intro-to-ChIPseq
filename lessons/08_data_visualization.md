---
title: "Visualization of peaks"
author: "Meeta Mistry"
date: "Thursday July 29th, 2017"
---

Approximate time: 45 minutes

## Learning Objectives
* Generate bigWig files
* Visualizing enrichment patterns at particular locations in the genome
* Use IGV to visualize BigWig, BED and data from ENCODE

## Visualization of ChIP-seq data

The first part of ChIP-sequencing analysis uses common processing pipelines, which involves the alignment of raw reads to the genome, data filtering, and identification of enriched signal regions (peak calling). In the second stage, individual programs allow detailed analysis of those peaks, biological interpretation, and visualization of ChIP-seq results.


There are various strategies for visualizing enrichment patterns and we will explore a few of them. To start, we will create bigWig files for our samples, a standard file format commonly used for ChIP-seq data visulaization.

### Creating bigWig files

The first thing we want to do is take our alignment files (BAM) and convert them into bigWig files. The bigWig format is an indexed binary format useful for dense, continuous data that will be displayed in a genome browser as a graph/track, but also is used as input for some of the visualization commands we will be running in `deepTools`. 

[`deepTools`](http://deeptools.readthedocs.org/en/latest/content/list_of_tools.html), is a suite of Python tools developed for the efficient analysis of high-throughput sequencing data, such as ChIP-seq, RNA-seq or MNase-seq. `deepTools` has a wide variety of commands that go beyond those that are covered in this lesson. We encourage you to look through the docuementation and explore on your own time. 


<img src="../img/bam_to_bigwig.png">

*Image aquired from [deepTools documentation](http://deeptools.readthedocs.io/en/latest/content/tools/bamCoverage.html?highlight=bigwig) pages*

Start an interactive session with 6 cores. *If you are already logged on to a compute node you will want to exit and start a new session*.

```bash
$ srun --pty -p short -t 0-12:00 --mem 8G -n 6 --reservation=HSPH bash
```

We will begin by creating a directory for the visualization output and loading the required modules to run `deepTools`.

```bash
$ cd ~/chipseq/results/
$ mkdir -p visualization/bigWig visualization/figures
```

```bash
$ module load gcc/6.2.0  python/2.7.12
$ module load deeptools/2.5.3 

```

To create our bigWig files there are two tools that can be useful: `bamCoverage` and `bamCompare`. The former will take in a single BAM file and return to you a bigWig file. The latter allows you to normalize two files to each other (i.e. ChIP sample relative to input).

Let's create a bigWig file for Nanog replicate 1 using the `bamCoverage` command. In addition to the input and output files, there are a few additional parameters we have added. 

* `normalizeTo1x`: Report read coverage normalized to 1x sequencing depth (also known as Reads Per Genomic Content (RPGC)). Sequencing depth is defined as: (total number of mapped reads * fragment length) / effective genome size). So the number provided here represents the effective genome size. Some examples for commonly used organisms can be [found here](http://deeptools.readthedocs.io/en/latest/content/feature/effectiveGenomeSize.html)
* `binSize`: size of bins in bases
* `smoothLength`: defines a window, larger than the `binSize`, to average the number of reads over. This helps produce a more continuous plot.
* `centerReads`: reads are centered with respect to the fragment length as specified by `extendReads`. This option is useful to get a sharper signal around enriched regions

```bash
bamCoverage -b bowtie2/H1hesc_Nanog_Rep1_chr12_aln.bam \
-o visualization/bigWig/H1hesc_Nanog_Rep1_chr12.bw \
--binSize 20 \
--normalizeTo1x 130000000 \
--smoothLength 60 \
--extendReads 150 \
--centerReads \
-p 6 2> ../logs/Nanog_rep1_bamCoverage.log
```

Now, if we wanted to create a bigWig file in which we normalize the ChIP against the input we would use `bamCompare`. The command is quite similar to `bamCoverage`, the only difference being you require two files as input (`b1` and `b2`).

```bash
bamCompare -b1 bowtie2/H1hesc_Nanog_Rep1_chr12_aln.bam \
-b2 bowtie2/H1hesc_Input_Rep1_chr12_aln.bam \
-o visualization/bigWig/H1hesc_Nanog_Rep1_chr12_bgNorm.bw \
--binSize 20 \
--normalizeTo1x 130000000 \
--smoothLength 60 \
--extendReads 150 \
--centerReads \
-p 6 2> ../logs/Nanog_rep1_bamCompare.log
```

> **NOTE:** When you are creating bigWig files for your full dataset, this will take considerably longer and you will not want to run this interactively (except for testing purposes). Instead, you might want to consider writing a job submission script with a loop that runs this command over all of your BAM files.

Since we are a subset of the data, using theses bigWigs for visualization would not give us meaningful results. As such, **we have created bigWig files from the full dataset that you can copy over and use for the rest of this lesson.**

```bash
cp /n/groups/hbctraining/chip-seq/full-dataset/bigWig/*.bw visualization/bigWig/

```




Compute the matrix (for TSS):
- plot Nanog on one plot (2 replicates)
- plot Pou5f1 one one plot (2 replicates)

```

```

- plot them separately each with a heatmap
- 


Compute matrix (for specific binding sites):
- look at Nanog enrichment on Pou5f1 bidning sites
- look at Pou5f1 enrichment on Nanog sites
- 









### Differential enrichment
To provide a more complex picture of biological processes in a cell, many studies aim to compare different datasets obtained by ChIP-seq. 








### IGV: Viewing files in a genome browser

* Copy over the bigWig files to your laptop using filezilla or scp. 
* Copy over the BEDtools overlap/intersect files to your computer.

* Start IGV and load the 2 rep1 files, and the overlap BED files. You will notice that there are positive and negative values on the track, what do you think this denotes in the context of normalization?

> You can generate a simple, non-normalized bigWig with `bamCoverage` and you won't see any negative values. 

* Now load the `Nanog_vs_Pou5f1_edgeR_sig.bed` and `Nanog_vs_Pou5f1_deseq2_sig.bed` (output of DiffBind, in your chipseq R project) into IGV.

* Finally, we are going to visually compare our output to the output from the full dataset from ENCODE, by loading that data from the IGV server.

***
*This lesson has been developed by members of the teaching team at the [Harvard Chan Bioinformatics Core (HBC)](http://bioinformatics.sph.harvard.edu/). These are open access materials distributed under the terms of the [Creative Commons Attribution license](https://creativecommons.org/licenses/by/4.0/) (CC BY 4.0), which permits unrestricted use, distribution, and reproduction in any medium, provided the original author and source are credited.*