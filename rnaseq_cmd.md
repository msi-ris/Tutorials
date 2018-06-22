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
You will need the following pieces of software to be installed on your local
computer to follow along with the tutorial.

- Terminal emulator
    - For Windows, use PuTTY (<https://www.putty.org/>)
        - Setup instructions [here](https://www.msi.umn.edu/support/faq/how-do-i-configure-putty-connect-msi-unix-systems).
    - For Mac, use Terminal, which is built-in
        - Go to Applications>Utilities>Terminal.app
    - For Linux, you should know how to get to your terminal emulator
- File transfer client
    - We recommend FileZilla (<https://filezilla-project.org/>) because it is
      reasonably powerful and easy to use.
    - Setup instructions [here](https://www.msi.umn.edu/support/faq/how-do-i-use-filezilla-transfer-data).
- Text editor
    - We will use KomodoEdit (<https://www.activestate.com/komodo-ide/downloads/edit>)
      because it allows us to edit text files on a remote machine. You can use
      a terminal-based editor if you so choose.

The software required for the actual RNASeq analysis is already installed on MSI
systems. We will show you how to access the analysis programs in later sections.

### UNIX (and Linux) Basics
MSI systems run GNU/Linux, which is a functional equivalent of commercial UNIX
(like Apple's OSX). The way that you will interact with Linux is very similar to
the way you would interact with a traditional UNIX system - through the command
line.

![Terminal]({{ "/graphics/rnaseq_cmd/terminal.png" | prepend: site.baseurl }})

Be warned, though, because the GNU/Linux versions of the programs behave
very similarly to the UNIX versions, but they are not identical. Always check
the help or manual pages if you experience some unexpected behavior from one
of the programs that you are running.

#### Common Commands
Throughout this tutorial, we will be using the command line to run programs that
analyze sequencing data. You will need to be familiar with navigating the MSI
directory structure and doing simple operations on files. Here are some of the
common commands that we will be using:

- `cd`: change directory. This command will move you around the directory
  structure on the cluster.
- `ls`: list files. This command will show you files and sub-directories that
  are contained within a specified location.
- `pwd`: print working directory. This command will show you the directory that
  you are currently working in.
- `cp`: copy. This command will copy files from one location to another.
- `mv`: move. This command will *move* files from one location to another.
- `rm`: remove. This command will delete files. Becareful, once they're gone,
  they're gone for good.
- `head`: show the "head" of a file. This command will show you the first few
  lines of a file.

#### Other Resources
There are plenty of resources available for learning how to interact with the
computer through the command line. For technical reference, the default
command line program on MSI, or shell, is called `bash`. Here are a few other
links for `bash` resources:

- [UCR Linux Basics](http://hpcc.ucr.edu/manuals_linux-basics_intro)
- [Advaned Bash Scripting Guide from The Linux Documentation Project](https://www.tldp.org/LDP/abs/html/)

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
