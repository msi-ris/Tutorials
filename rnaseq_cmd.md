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
    - For Windows, use PuTTY (<https://www.putty.org/>).
        - Setup instructions [here](https://www.msi.umn.edu/support/faq/how-do-i-configure-putty-connect-msi-unix-systems).
    - For Mac, use Terminal, which is built-in.
        - Go to Applications>Utilities>Terminal.app
    - For Linux, you should know how to get to your terminal emulator already.
- File transfer client
    - We recommend FileZilla (<https://filezilla-project.org/>) because it is
      reasonably powerful and easy to use.
    - Setup instructions [here](https://www.msi.umn.edu/support/faq/how-do-i-use-filezilla-transfer-data).
- Text editor
    - We will use Komodo Edit (<https://www.activestate.com/komodo-ide/downloads/edit>)
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
- `ssh`: secure shell. This will let you log in to a remote machine to execute
  commands. We will use this command to log in to MSI systems.

#### Other Resources
There are plenty of resources available for learning how to interact with the
computer through the command line. For technical reference, the default
command line program on MSI, or shell, is called `bash`. Here are a few other
links for `bash` resources:

- [UCR Linux Basics](http://hpcc.ucr.edu/manuals_linux-basics_intro)
- [Advaned Bash Scripting Guide from The Linux Documentation Project](https://www.tldp.org/LDP/abs/html/)

### Accessing MSI
We will now log in to MSI systems. The first host we will access is called the
`login` node. **Don't run computationally intense tasks on this node.** You
will make it hard for other uses to access the systems by reducing the
performance of the login gateway, and we will yell at you. A lot.

On Windows, open PuTTY, and follow the instructions at the PuTTY setup link
given in the previous section. You can skip step 5, because we will just connect
to the login node.

For Mac or Linux users, type `ssh X.500@login.msi.umn.edu` and press enter.
Replace the `X.500` with your UMN X.500 ID. It will most likely ask you for a
password. Enter your UMN internet ID password, and press enter. You won't be
able to see the password as it's entered. If it goes well, the server will
give you a message-of-the-day, confirming that you have successfully logged in
to the system.

![SSH]({{ "/graphics/rnaseq_cmd/ssh.png" | prepend: site.baseurl }})
![Login]({{ "/graphics/rnaseq_cmd/login.png" | prepend: site.baseurl }})


## Part 1: RNASeq Overview
### Why RNASeq
- expression
- variant discovery without reference genome
- reduced representation, genes are often the most interpretable pieces of a genome

### Common Genomics File Formats
RNASeq, being a genomics technique, uses standard file formats for genomics
analysis. This is not an exhaustive list, but should introduce the file types
that we will use in this tutorial.

- FASTA

    Holds sequence information, without any associated quality information. The
    FASTA format has a sequence name, denoted with `>` followed by the name of
    the sequence. The next line(s) have the sequences. The file may have one or
    more sequence records.

    ```
    >Sequence 1
    ATCGA...
    >Sequence 2
    GGTAC...
    ```

    Reference sequences are generally stored in the FASTA format.
- FASTQ

    Holds sequence information with associated quality scores. Each FASTQ record
    has four lines: a name, a nucleotide sequence, a comment, and a quality
    string. The name starts with `@` and the comment starts with `+`. The
    quality scores are stored as ASCII characters, treating each character's
    decimal value as the quality score (with a +33 offset\*). You will not have
    to read a FASTQ directly - the alignment programs will understand them.
    Quality is defined as -log10(P that the base is called incorrectly), which
    is the Phred scale used in Sanger sequencing traces.

    \*: Note that older FASTQ files may have a +64 offset. The quality control
    program we will use in this tutorial will make an educated guess as to the
    proper offset.

    ```
    @D00635:256:CB8P5ANXX:1:1108:2215:2225 1:N:0:ATTACTCG+TATAGCCT
    GCGCTAAAGCAGTGATTAGAGGGAAATTTATAGCACCAAATGCCCACAGAA
    +
    @B=BBGGGGCBDDC@GGEGGGGGGGGGGGGGFGGGEGGGGGGGGGGGGG>D
    @D00635:256:CB8P5ANXX:1:1108:2547:2188 1:N:0:ATTACTCG+TATAGCCT
    CCGCACATCACAAACGTGATTCTGCGAATGCTTCTGACTAGTTTTTGTCGG
    +
    BBBCCEGGGCGGGGGGGGGGGGCGGGGGGGGGGGGGGGGG>F>F>GGGGGG
    ```

    Reads from the sequencing instrument are generally stored in the FASTQ
    format.
- SAM/BAM

    Holds alignment information. While sequence alingments can technically be
    stored in FASTA format, it is very inefficient for genomics datasets.
    Additionally, genomics alignments tend to be a series of pairwise alignments
    to a reference sequence, and not a multiple sequence alignment.

    A SAM (**S**equence **A**lignment/**M**ap) file stores positon, read
    sequence, mismatch information, and alignment confidence scores, among other
    information. A BAM file is a binary representation of a SAM file, which is
    machine-readable and much smaller on disk. You generally will not have to
    parse a SAM/BAM file on your own; most of the tools you will encounter will
    know how to perform operations on them.

    A link to the SAM/BAM specification is [here](https://samtools.github.io/hts-specs/SAMv1.pdf).
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
