---
layout: default
title: RNASeq Analysis With the Command Line
permalink: /rnaseq_cmd/
exclude: false
updated: 2018-06-22
delivered: NA
---

# {{page.title}}
*Last Updated: {{page.updated}}*

*Last Delivered: {{page.delivered}}*


## Part 0: Introduction
### Goals
- Learn some fundamental bash commands for interacting with MSI systems
- Log in to MSI systems and access a high-performance compute node
- Become familiar with several core file formats of genomics
- Learn the basics of RNASeq data analysis, including read mapping, expression
  counts, and testing for differential expression

### Required Software
### UNIX Basics
- common commands
- other resources (TLDP ABS)

### Intro to MSI
- login (screenshots of puTTy and Terminal)
- HPC and queues

## Part 1: RNASeq Overview
### Why RNASeq
- expression
- variant discovery without reference genome
- reduced representation, genes are often the most interpretable pieces of a genome

### Common Genomics File Formats
- FASTA/FASTQ
- SAM/BAM
- GTF/GFF

### Overall Workflow
- cartoon
- Numbered list of steps

## Part 2: FASTQ Quality Control
- isub
- copy tutorial files
- load fastqc module
- run fastqc on all files
- copy to local machine, look at them

## Part 3: Cleaning Reads
- load trimmomatic module
- run trimmomatic on all files
- run fastqc on cleaned files, check again

## Part 4: Alignment to Genome
- load hisat2 module
- give path to pre-built genome index, but show how to do it for later
- align cleaned reads with hisat2

## Part 5: Counts
- load htseq module
- Produce counts matrix
- view it with less to see what the format is

## Part 6: DEG Analysis
- EdgeR or DESeq2
- filter matrix
- normalize values
- identify differentially expressed genes
- make summary plots (PCA, heatmap with dendrogram, ma-plot)

## Part 7: Recommendations
- design experiments with power (controls and replicates)
- be careful with public data (cf rnalater)
- "best-in-class" software
  - HISAT2
  - Kallisto (not generally)
  - HTSeq
  - EdgeR, DESeq2
- Be careful with tools built for microarrays

## Part 8: Other RNA Analyses
- Coexpression (WGCNA)
- Tx assembly (including splicing)
- Variant discovery (in absence of reference genome)
