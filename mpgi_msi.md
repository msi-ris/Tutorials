---
layout: default
title: MPGI-MSI Workshop
permalink: /mpgi_msi/
exclude: false
updated: 2018-12-25
delivered: NA
---

<div id="navdiv" markdown="1">

## Table of Contents
- [Introduction](#0)
    - [Formatting in This Document](#0.1)
    - [Goals](#0.2)
    - [Scope of the Tutorial](#0.3)
- [Module 1: Intro to MSI](#1)
- [Module 2: Intro to Genomics](#2)
- [Module 3: Intro to PBS/Jobs](#3)
- [Module 4: Parallelization](#4)

</div>

# <a name="top"></a> {{page.title}}
*Last Updated: {{page.updated}}*  
*Last Delivered: {{page.delivered}}*

## <a name="0"></a>Part 0: Introduction
### <a name="0.1"></a>0.1 Formatting in This Document
Throughout this tutorial, there will be formatting cues to highlight various
pieces of information.

<div class="info" markdown="1">

This is just background information. There are no tutorial-related tasks in
these boxes. Links to supporting material and further explanations of points we
raise in the tutorial will appear like this.

</div>

<div class="warn" markdown="1">

This is a warning. Common pitfalls, cautionary inforation, and important points
to consider will appear like this.

</div>

```
This is code, or a literal value that you must enter or select to run a part
of the tutorial
```

Commands that are to be entered on the command-line will begin with `%`. Do not
re-enter the `%` character when you type the commands into your prompt.

[Return to top](#top)
### <a name="0.2"></a> 0.2: Goals
- Learn some fundamental bash commands for interacting with MSI systems
- Log in to MSI systems and access a high-performance compute node
- Become familiar with several core file formats of genomics
- Learn the basics of RNASeq data analysis, including read mapping, expression
  counts, and testing for differential expression
- Learn how to use the UMII-RIS pipelines to perform a differential expression
  analysis with RNAseq

[Return to top](#top)
### <a name="0.3"></a> 0.3: Scope of the Tutorial
This tutorial will only cover differential gene expression analysis of
**bulk RNAseq** of mRNA. We will not cover single cell RNAseq analysis or small
RNA sequencing analysis. We will also not cover coexpression analysis or
transcriptome assembly in detail. There are links to guides at the bottom of the
tutorial document for coexpression analysis and transcriptome assembly.

While we will be teaching how to use analysis tools from the command-line, the
names and options that we are supplying should be available in the Galaxy
versions. See our [Illumina QC With Galaxy](https://pages.github.umn.edu/MSI-RIS/Tutorials/illumina_galaxy/)
tutorial for details on how to create workflows with Galaxy that will let you
run similar analyses that we will cover here.

[Return to top](#top)
### <a name="0.4"></a> 0.4: Required Software
You will need the following pieces of software to be installed on your local
computer to follow along with the tutorial.

- Terminal emulator
    - For Windows, use PuTTY (<https://www.putty.org/>).
        - Setup instructions [here](https://www.msi.umn.edu/support/faq/how-do-i-configure-putty-connect-msi-unix-systems).
    - For Mac, use Terminal, which is built-in.
        - Go to Applications>Utilities>Terminal.app
    - For Linux workstations in the MSI computer lab, go to
      Applications>System Tools>Terminal. The Applications menu is in the
      top-left of the desktop screen.
        - For personal Linux workstations, please refer to your distribution/desktop notes
- File transfer client
    - We recommend FileZilla (<https://filezilla-project.org/>) because it is
      reasonably powerful and easy to use.
    - Be sure to download the **client** program, and not the server program.
    - Setup instructions [here](https://www.msi.umn.edu/support/faq/how-do-i-use-filezilla-transfer-data).

The software required for the actual RNASeq analysis is already installed on MSI
systems. We will show you how to access the analysis programs in later sections.

<div class="info" markdown="1">

The software that we use for the bulk RNAseq analysis are as follows:

- FastQC for quality control of reads
- Trimmomatic for quality and adapter trimming
- HISAT2 for read mapping
- SAMTools for sorting and filtering
- Picard Tools for insert size metric calculation
- featureCounts (from subread) for generating expression counts
- edgeR (in R) for differential expression testing
- Rmarkdown (in R) for report generation

</div>

You will not be required to call any of these programs directly for the
tutorial, but if you would like to implement the same pipeline outside of MSI,
you will need those tools available.

[Return to top](#top)

### <a name="0.5"></a> 0.5: Accessing MSI
We will now log in to MSI systems. The first host we will access is called the
`login` node.

<div class="warn" markdown="1">

Don't run computationally intense tasks on this node. Other users will not be
able to access the systems because the login nodes will become unresponsive.
Doing this will result in disciplinary action, and repeats will result in
account restrictions.

</div>

On Windows, open PuTTY, and follow the instructions at the PuTTY setup link
given in the previous section. You can skip step 5, because we will just connect
to the login node.

For Mac or Linux users, type `ssh X.500@login.msi.umn.edu` and press enter.
Replace the `X.500` with your UMN X.500 ID. It will most likely ask you for a
password. Enter your UMN Internet ID password, and press enter. You won't be
able to see the password as it's entered. If it goes well, the server will
give you a message-of-the-day, confirming that you have successfully logged in
to the system.

![SSH]({{ "/graphics/rnaseq_cmd/ssh.png" | prepend: site.baseurl }})
![Login]({{ "/graphics/rnaseq_cmd/login.png" | prepend: site.baseurl }})

[Return to top](#top)

#### 0.5.1: Common Commands
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
- `rm`: remove. This command will delete files. Be careful, once they're gone,
  they're gone for good.
- `head`: show the "head" of a file. This command will show you the first few
  lines of a file.
- `ssh`: secure shell. This will let you log in to another machine to execute
  commands. We will use this command to connect to a high-performance computing
  login node from the main login nodes.

## <a name="0.5.2"></a> 0.5.2: Practice Commands
To get familiar with the commands that we will be using in this tutorial, we
will walk through a few commands that will help us set up the directories to run
the pipeline we are teaching in this tutorial.

1. Make a directory in global scratch for your files. We recommend organizing
   your files in scratch in such a way that you can easily find your files and
   do not clash with other users. Global scratch is a public space, so it is
   helpful to be organized there. Use your X.500 ID instead of `YOUR_USER_NAME`
   in the following command:

   ```bash
   % mkdir /scratch.global/YOUR_USER_NAME
   ```

   If you have already made this directory, then you will get an error saying
   that it already exists. You can use the `-p` option to `mkdir` to disable
   that behavior:

   ```bash
   % mkdir -p /scratch.global/YOUR_USER_NAME
   ```

2. Navigate to the directory you just made with the `cd` command:

   ```bash
   % cd /scratch.global/YOUR_USER_NAME
   ```

3. Verify that you got to the correct directory with the `pwd` command. `pwd`
   will print your current working directory to the terminal.

   ```bash
   % pwd
   /scratch.global/konox006
   ```

   You should see your username instead of `konox006`.

4. Make output and working directories in your scratch directory. The output
   directory will hold the final results of the pipeline and the working
   directory will hold the intermediate files.

   ```bash
   % mkdir RNAseq_Tutorial_Out
   % mkdir RNAseq_Tutorial_Work
   ```

   Like the previous `mkdir` command, you can also supply the `-p` option if
   they already exist:

   ```bash
   % mkdir -p RNAseq_Tutorial_Out
   % mkdir -p RNAseq_Tutorial_Work
   ```

5. Verify that the directories were made with the `ls` command. `ls` lists files
   and directories that are contained in a specified location. If you run `ls`
   without any arguments, then your current working directory is used:

   ```bash
   % ls
   RNASeq_Tutorial_Out  RNAseq_Tutorial_Work
   ```

   If you have been using your scratch directory, then you may see additional
   files there.

6. Just for illustrative purposes, let's copy the gene annotations file from the
   tutorial directory into the scratch directory. Verify that it has been
   copied, too, using `ls`. Be sure to put the dot (`.`) as the final argument
   to the `cp` command.

   ```bash
   % cp /home/msistaff/public/RNAseq_Tutorial/Reference/Annotations.gtf.gz .
   % ls
   Annotations.gtf.gz  RNASeq_Tutorial_Out  RNAseq_Tutorial_Work
   ```

   <div class="info" markdown="1">
    The dot (`.`) is a bash shorthand for the current working directory when
    used as an argument to a command.
    </div>

7. Then, you can delete it with the `rm` command.

   ```bash
   % rm Annotations.gtf.gz
   % ls
   RNASeq_Tutorial_Out  RNAseq_Tutorial_Work
   ```

[Return to top](#top)

<div class="info" markdown="1">

#### 0.6.1: Other Resources
There are plenty of resources available for learning how to interact with the
computer through the command line. For technical reference, the default
command line program on MSI, or shell, is called `bash`. Here are a few other
links for `bash` resources:

- [UCR Linux Basics](http://hpcc.ucr.edu/manuals_linux-basics_intro)
- [Advanced Bash Scripting Guide from The Linux Documentation Project](https://www.tldp.org/LDP/abs/html/)

</div>

[Return to top](#top)

## <a name="1"></a> Part 1: RNASeq Overview

<div class="info" markdown="1">

### <a name="1.1"></a> 1.1: Why RNASeq
RNAseq is name for a high-throughout sequencing technique that targets the
transcribed portions of the genome. For this tutorial, we will be focusing on
the analysis of **bulk RNAseq** data. This type of data involves extraction of
messenger RNA (mRNA) from whole tissue, organ, or organism samples. This
technique is often applied for studies of gene expression, but can also be
used to assemble transcript sequences or identify genetic variants (see
[**Part 8**](#8) at the end of this document for more information).

Bulk RNAseq is a useful tool to identify the transcriptomic response to a
disease or stress condition, changes in expression during development of an
organism, or differences in expression among tissue types. The transcribed
gene regions are often the most interpretable regions of the genome, because
changes in mRNA content are correlated with changes in protein content (though,
not always).

[Return to top](#top)
### <a name="1.2"></a> 1.2: Common Genomics File Formats
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

    Holds alignment information. While sequence alignments can technically be
    stored in FASTA format, it is very inefficient for genomics datasets.
    Additionally, genomics alignments tend to be a series of pairwise alignments
    to a reference sequence, and not a multiple sequence alignment.

    A SAM (**S**equence **A**lignment/**M**ap) file stores position, read
    sequence, mismatch information, and alignment confidence scores, among other
    information. A BAM file is a binary representation of a SAM file, which is
    machine-readable and much smaller on disk. You generally will not have to
    parse a SAM/BAM file on your own; most of the tools you will encounter will
    know how to perform operations on them.

    ```
    @HD VN:1.6 SO:coordinate
    @SQ SN:ref LN:45
    r001 99   ref 7  30 8M2I4M1D3M = 37 39  TTAGATAAAGGATACTG *
    r002 0    ref 9  30 3S6M1P1I4M * 0  0   AAAAGATAAGGATA    *
    r003 0    ref 9  30 5S6M       * 0  0   GCCTAAGCTAA       * SA:Z:ref,29,-,6H5M,17,0;
    r004 0    ref 16 30 6M14N5M    * 0  0   ATAGCTTCAGC       *
    r003 2064 ref 29 17 6H5M       * 0  0   TAGGC             * SA:Z:ref,9,+,5S6M,30,1;
    r001 147  ref 37 30 9M         = 7  -39 CAGCGGCAT         * NM:i:1
    ```

    A link to the SAM/BAM specification is [here](https://samtools.github.io/hts-specs/SAMv1.pdf).
- GTF/GFF (GFF3)

    Holds genomic feature annotations, like gene models. These formats are
    somewhat loosely defined, but files from databases such as NCBI or
    Ensembl should be handled well by genomics analysis programs. The
    information stored in a GTF or GFF includes each feature's boundaries,
    type (gene, CDS, TF binding site, etc), strand, name, and any parent/child
    relationships. For example, a `mRNA` feature may be the child of a `gene`
    feature, and may have several `CDS` and `exon` children features.

    The GFF and GTF specifications can be found [here](https://useast.ensembl.org/info/website/upload/gff.html).

</div>

[Return to top](#top)
### <a name="1.3"></a> 1.3: Overall Workflow
Here is a schematic of the workflow that we will follow for this tutorial. The
pieces of data are shown in rectangles, and the software are shown in rounded
bubbles with dashed borders. The final output is shown in red.

![Workflow]({{ "/graphics/rnaseq_cmd/workflow.png" | prepend: site.baseurl }})

The steps of the workflow are as follows:

1. Summarize read quality
2. Clean reads for low-quality bases and adapter contaminants
3. Map reads to genome
4. Count reads within genes
5. Filter counts for genes with low expression
6. Test for differential expression

[Return to top](#top)
## <a name="2"></a> Part 2: Running an Analysis With `CHURP`
The workflow steps outlined in the previous section are implemented in a package
developed and maintained by the University of Minnesota Informatics Institute
and the Research Informatics Solutions groups. This software package is
available on MSI systems as a software module.

You may also download the latest version of the code from the UMN GitHub
instance: <https://github.umn.edu/MSI-RIS/CHURP>. This will give you greater
flexibility, because you will be able to modify the underlying Python and shell
scripts. However, we offer limited support for workflows that use modified
versions of our pipelines.

[Return to top](#top)
### <a name="2.1"></a> 2.1: Prepare to Run `CHURP`
First, connect to a login node on Mesabi by typing `ssh mesabi` at the login
prompt. If you do no have ssh keys set up, you will have to enter your password
again. It is the same as your X.500 password. Note the `@login` part of the
prompt changes to reflect that you have started a shell on the Mesabi login
node.

![Mesabi]({{ "/graphics/rnaseq_cmd/mesabi.png" | prepend: site.baseurl }})

<div class="warn" markdown="1">

Do not run computationally-intense tasks on this node, either; this is a login
node for the compute cluster. Running analysis tasks on the login node for the
cluster will deplete the resources for other users to connect and get their
work done, and will result in account restrictions.

That said, running the pipelines as described in this tutorial does not count as
a computationally-intense task; it properly submits tasks to the job scheduler.

</div>

From here on, the commands and their output will be shown in text, rather than
in screenshots of terminal emulators.

Next, load the `python3` module. This is required to run `CHURP`. Navigate to
the `/home/msistaff/public/RNAseq_Tutorial/CHURP` directory with `cd`. Run the
main control script, `churp.py`.

```bash
% module load python3
% cd /home/msistaff/public/RNAseq_Tutorial/CHURP
% python churp.py
Usage: churp.py <subcommand> <options>

where <subcommand> is the name of the pipeline that is to be run. The specified
<options> will be applied to the operations in the pipeline. Each pipeline has
its own set of options that must be specified. To see a full listing of each
available option for a given pipeline, pass the '--help' option. Alternately,
online help is maintained at the GitHub repository.

Currently, the following subcommands are supported:
    - group_template
    - bulk_rnaseq

For issues, contact help@msi.umn.edu.
Version: 0.0.1
2018-11-04
```

If you get a message similar to the one above, then you are ready to run the
pipelines.

[Return to top](#top)
### <a name="2.2"></a> 2.2: Input Data for the Pipelines
The pipelines requires that the following pieces of data be available:

- Directory of FASTQ files from the UMGC
- Reference genome indexed with HISAT2
- Gene annotations in GTF format

For this tutorial, we have provided these data. The FASTQ files are located at
`/home/msistaff/public/RNAseq_Tutorial/Reads`. The
indexed reference genome is located at
`/home/msistaff/public/RNAseq_Tutorial/Reference/GRCm38_19`. The accompanying
GTF gene annotations are located at
`/home/msistaff/public/RNAseq_Tutorial/Reference/Annotations.gtf.gz`.

When you run your own analysis, you may need to generate your own HISAT2 index
for your reference genome. We will provide some brief instructions in a later
section of this tutorial document, but the best source of instructions is from
the HISAT2 manual.

[Return to top](#top)
### <a name="2.3"></a> 2.3: Running the Pipeline
The subcommand that we will be using in this tutorial is the `bulk_rnaseq`
subcommand. Provide it as an argument after the `churp.py` command. The default
help message tells you the bare minimum arguments that are required to analyze
the data. To see a full help message, add the `-h` option after the 
`bulk_rnaseq` subcommand. For more detail on the options that are available
for the `bulk_rnaseq` pipeline, you can access the latest version of the manual
at the [GitHub Wiki page](https://github.umn.edu/MSI-RIS/CHURP/wiki/bulk_rnaseq).

```bash
% python churp.py bulk_rnaseq
----------
Thank you for using the refactor of gopher-pipelines. This software was
developed by the Research Informatics Solutions (RIS) group at MSI with funding
from the University of Minnesota Informatics Institute (UMII). For help, please
contact help@msi.umn.edu.

https://www.msi.umn.edu/
https://research.umn.edu/units/umii

----------
ERROR


You did not specify sufficient options to run the bulk_rnaseq subcommand of
CHURP. You must specify a FASTQ directory (-f). Additionally, you
must either specify a path to a HISAT2 index (-x) and GTF (-g), or an organism
name (-r). If you are building a group template file, you need only specify a
FASTQ directory. Please fix your command line and re-run.

```

As the help message shows, you must specify a FASTQ directory, a HISAT2 index,
and a GTF file. We have provided these for you fur the tutorial, but before we
actually run the analysis, through, we have to generate experimental group files
for differential expression testing.

[Return to top](#top)
#### <a name="2.3.1"></a> 2.3.1: Generating Experimental Groups
For differential expression testing, we have to generate a file that describes
which samples are part of which experimental group. We do this with the
`group_template` subcommand. This particular subcommand takes an additional
argument. In this case, we will supply `bulk_rnaseq` because we are generating
a groups file for the `bulk_rnaseq` pipeline. We supply the FASTQ directory as
the argument.

*Please note that the backslashes `\` and new lines in the following commands
are only for readability purposes. You don't have to reproduce the same
indentation structure as the tutorial document! It should still work if you do
reproduce the structure, though.*

```bash
% python churp.py group_template bulk_rnaseq \
    -f /home/msistaff/public/RNAseq_Tutorial/Reads \
    -o /scratch.global/YOUR_USER_NAME/RNAseq_Tutorial_Out/Groups.csv
----------
Thank you for using the refactor of gopher-pipelines. This software was
developed by the Research Informatics Solutions (RIS) group at MSI with funding
from the University of Minnesota Informatics Institute (UMII). For help, please
contact help@msi.umn.edu.

https://www.msi.umn.edu/
https://research.umn.edu/units/umii

----------
SUCCESS

Template file: /scratch.global/konox006/RNAseq_Tutorial_Out/Groups.csv

All sample groups and any additional columns have specified have been filled
with NULL. Please edit the file and write in the correct values for your
dataset. Samples with the same "Group" label will be treated as replicates
in the analysis. Samples with a "Group" value of NULL will not be used in
downstream differential expression analysis. When you have edited the file to
your liking, supply its path to the "bulk_rnaseq" pipeline with the -e
option to enable group testing.

```

The template file must then be edited to contain the assignments to each group.
By default, all samples are listed as being part of a group labeled `NULL`. The
default file is shown below:

```
SampleName,Group
BoneMarrow-1,NULL
BoneMarrow-2,NULL
BoneMarrow-3,NULL
BoneMarrow-4,NULL
Spleen-1,NULL
Spleen-2,NULL
Spleen-3,NULL
Spleen-4,NULL
```

Use the `nano` text editor (or your favorite terminal editor program) to assign
the samples to their groups. In this case, the first four samples are part of
group `BoneMarrow` and the final four samples are part of group `Spleen`.

```bash
% nano /scratch.global/YOUR_USER_NAME/RNAseq_Tutorial_Out/Groups.csv
```

The window will display the contents of the CSV file. Use the arrow keys to
move the cursor around in the window to replace text. Edit the file so that it
looks like the file below:

```
SampleName,Group
BoneMarrow-1,BoneMarrow
BoneMarrow-2,BoneMarrow
BoneMarrow-3,BoneMarrow
BoneMarrow-4,BoneMarrow
Spleen-1,Spleen
Spleen-2,Spleen
Spleen-3,Spleen
Spleen-4,Spleen
```

Once you are done editing the file, press `[Control]` and `O` (letter 'o') to
save the file. It will ask you the file name; leave the default value to
overwrite the CSV. Press `[Enter]` to confirm saving the file. Then press
`[Control]` and `X` to exit. This is a manual step because the researcher must
made a decision as to which groups should be compared for differential
expression testing.

[Return to top](#top)
#### <a name="2.3.2"></a> 2.3.2: Submitting the Pipeline Jobs
Now that we have assigned the samples to groups, we will run the `bulk_rnaseq`
pipeline, supplying the file that contains the group assignments. This is done
by supplying the `-e` option in addition to the `-f`, `-x`, and `-g` options. We
also supply the `--submit` option to automatically send the jobs to the queue.
If you omit the `--submit` option, then you will have a chance to edit the
pipeline script and the samplesheet. The formats for these files will be
described later in the document.

<div class="warn" markdown="1">

We are using the `--unstranded` option here because the source of the data is
from a lab that used an unstranded library prep protocol (NEBNext Ultra RNA
kit). The default option is for reverse-stranded libraries, which is the
standard kit used by the UMGC (TruSeq Stranded RNA).

</div>

For the sake of getting the jobs completed more quickly, we will decrease the
size of the resource request.

```bash
% python churp.py bulk_rnaseq \
    -e /scratch.global/YOUR_USER_NAME/RNAseq_Tutorial_Out/Groups.csv \
    -f /home/msistaff/public/RNAseq_Tutorial/Reads \
    -x /home/msistaff/public/RNAseq_Tutorial/Reference/GRCm38_19 \
    -g /home/msistaff/public/RNAseq_Tutorial/Reference/Annotations.gtf.gz \
    -o /scratch.global/YOUR_USER_NAME/RNAseq_Tutorial_Out \
    -d /scratch.global/YOUR_USER_NAME/RNAseq_Tutorial_Work \
    --unstranded \
    --ppn 4 --mem 12000 -w 2 --submit
----------
Thank you for using the refactor of gopher-pipelines. This software was
developed by the Research Informatics Solutions (RIS) group at MSI with funding
from the University of Minnesota Informatics Institute (UMII). For help, please
contact help@msi.umn.edu.

https://www.msi.umn.edu/
https://research.umn.edu/units/umii

----------
SUCCESS

Your pipeline has been submitted. For reference, the pipeline script and the
samplesheet are given at the paths below:

Pipeline script: /scratch.global/konox006/[...]bulk_rnaseq.pipeline.sh
Samplesheet: /scratch.global/konox006/[...]bulk_rnaseq.samplesheet.txt

Below is the output from qsub.
Qsub stdout:
Output and logs will be written to /scratch.global/konox006/RNAseq_Tutorial_Out
Emails will be sent to konox006@umn.edu
Single samples job array ID: 8701303[].mesabim3.msi.umn.edu
Summary job ID: 8701304.mesabim3.msi.umn.edu

Qsub stderr:

```

<div class="warn" markdown="1">

The default locations of the output and working directories is in global scratch
space. If you want to save any of the files from your analysis, be sure to
copy them into your group's home directory. MSI removes data from global scratch
that is more than 30 days old. If you leave data there, then it will not be
available for long-term reference.

</div>

The message that prints tells you the IDs of the submitted jobs and the
address to which the email notifications will be sent. For your reference, the
pipeline script and samplesheet paths are also printed. If you would like to
reproduce the analysis, you must keep both the pipeline script and the
samplesheet.

*Note that the paths to the pipeline script and the samplesheet have been
truncated in this webpage for readability. When you run the pipeline in the
terminal, you will see the full path printed.*

You can verify that the jobs are submitted by issuing the `qstat -t` command.
The `-t` expands the job array that we use to process samples in batches.

```bash
% qstat -t
Job ID                    Name             User            Time Use S Queue
------------------------- ---------------- --------------- -------- - -----
8840519[1].mesabim3.msi.umn.e ..._sample.pbs-1 konox006               0 Q small
8840519[2].mesabim3.msi.umn.e ..._sample.pbs-2 konox006               0 Q small
8840519[3].mesabim3.msi.umn.e ..._sample.pbs-3 konox006               0 Q small
8840519[4].mesabim3.msi.umn.e ..._sample.pbs-4 konox006               0 Q small
8840519[5].mesabim3.msi.umn.e ..._sample.pbs-5 konox006               0 Q small
8840519[6].mesabim3.msi.umn.e ..._sample.pbs-6 konox006               0 Q small
8840519[7].mesabim3.msi.umn.e ..._sample.pbs-7 konox006               0 Q small
8840519[8].mesabim3.msi.umn.e ..._sample.pbs-8 konox006               0 Q small
8840520.mesabim3.msi.umn.edu ...ary_stats.pbs konox006               0 H small
```

The `Q` means that the job is queued. When the jobs begins to run, it will turn
to `R`. When it finishes, it will become `C`. The `H` means that the job is
held - we set this job to wait until the previous jobs complete successfully
before marking it as eligible to run. The `H` will eventually become `Q`, and
then `R` and `C`. After a while, the job will be cleared from the list.

[Return to top](#top)
#### <a name="2.3.3"></a> 2.3.3: Viewing Results
Once all of the jobs have finished (all are listed with `C` status, or are
no longer in the output of `qstat -t`), you will be able to view the results.
The output is in the directory that was given as the output directory in the
pipeline output message. For our example, this is
`/scratch.global/YOUR_USER_NAME/RNAseq_Tutorial_Out`.

Navigate to the output directory and list the contents:

```bash
% cd /scratch.global/YOUR_USER_NAME/RNAseq_Tutorial_Out
% ls -lF
total 5.1M
-rw-rw---- 1 konox006 msistaff 1.5K Nov  6 10:01 2018-11-06.konox006.bulk_rnaseq.pipeline.sh
-rw-rw---- 1 konox006 msistaff 4.9K Nov  6 10:01 2018-11-06.konox006.bulk_rnaseq.samplesheet.txt
lrwxrwxrwx 1 konox006 msistaff   56 Nov  6 10:07 allsamples_work_directory
-> /scratch.global/konox006/RNAseq_Tutorial_Work/allsamples/
-r--r----- 1 konox006 msistaff 732K Nov  6 10:07 Annotations.gtf.gz
-rw-rw---- 1 konox006 msistaff 2.5M Nov  6 10:07 Bulk_RNAseq_Report.html
-rw-rw---- 1 konox006 msistaff 6.6K Nov  6 10:04 bulk_rnaseq_single_sample.pbs.e8840519-1
-rw-rw---- 1 konox006 msistaff 6.6K Nov  6 10:05 bulk_rnaseq_single_sample.pbs.e8840519-2
-rw-rw---- 1 konox006 msistaff 6.6K Nov  6 10:04 bulk_rnaseq_single_sample.pbs.e8840519-3
-rw-rw---- 1 konox006 msistaff 6.6K Nov  6 10:04 bulk_rnaseq_single_sample.pbs.e8840519-4
-rw-rw---- 1 konox006 msistaff 6.3K Nov  6 10:05 bulk_rnaseq_single_sample.pbs.e8840519-5
-rw-rw---- 1 konox006 msistaff 6.3K Nov  6 10:04 bulk_rnaseq_single_sample.pbs.e8840519-6
-rw-rw---- 1 konox006 msistaff 6.3K Nov  6 10:04 bulk_rnaseq_single_sample.pbs.e8840519-7
-rw-rw---- 1 konox006 msistaff 6.3K Nov  6 10:04 bulk_rnaseq_single_sample.pbs.e8840519-8
-rw-rw---- 1 konox006 msistaff  493 Nov  6 10:03 bulk_rnaseq_single_sample.pbs.o8840519-1
-rw-rw---- 1 konox006 msistaff  493 Nov  6 10:03 bulk_rnaseq_single_sample.pbs.o8840519-2
-rw-rw---- 1 konox006 msistaff  493 Nov  6 10:03 bulk_rnaseq_single_sample.pbs.o8840519-3
-rw-rw---- 1 konox006 msistaff  493 Nov  6 10:03 bulk_rnaseq_single_sample.pbs.o8840519-4
-rw-rw---- 1 konox006 msistaff  453 Nov  6 10:03 bulk_rnaseq_single_sample.pbs.o8840519-5
-rw-rw---- 1 konox006 msistaff  453 Nov  6 10:03 bulk_rnaseq_single_sample.pbs.o8840519-6
-rw-rw---- 1 konox006 msistaff  453 Nov  6 10:03 bulk_rnaseq_single_sample.pbs.o8840519-7
-rw-rw---- 1 konox006 msistaff  453 Nov  6 10:03 bulk_rnaseq_single_sample.pbs.o8840519-8
drwxrwx--- 2 konox006 msistaff 4.0K Nov  6 10:05 Coordinate_Sorted_BAMs/
drwxrwx--- 2 konox006 msistaff 4.0K Nov  6 10:07 Counts/
drwxrwx--- 2 konox006 msistaff 4.0K Nov  6 10:07 DEGs/
-rw-rw---- 1 konox006 msistaff  29K Nov  6 10:07 gene_id_gene_name_map.txt
-rw-rw---- 1 konox006 msistaff  177 Nov  6 10:01 Groups.csv
drwxrwx--- 2 konox006 msistaff 4.0K Nov  6 10:05 InsertSizeMetrics/
drwxrwx--- 2 konox006 msistaff 4.0K Nov  6 10:06 Logs/
drwxrwx--- 2 konox006 msistaff 4.0K Nov  6 10:07 Plots/
-rw-rw---- 1 konox006 msistaff  14K Nov  6 10:07 run_summary_stats.pbs.e8840520
-rw-rw---- 1 konox006 msistaff 9.0K Nov  6 10:07 run_summary_stats.pbs.o8840520
lrwxrwxrwx 1 konox006 msistaff   59 Nov  6 10:07 singlesamples_work_directory
-> /scratch.global/konox006/RNAseq_Tutorial_Work/singlesamples/
```

The `-l` option to `ls` gives "long format" which includes permissions, owner,
group, and modification times. The `-F` option displays  characters at the end
of names that signify the type of object it is, like `/` at the end of the names
of directories.

There are multiple levels types of output that get generated:

- PBS output files
- Insert size metrics (if data are paired-end)
- Expression counts
- Summary plots
- Differentially expressed genes (if groups were specified)
- Per-sample log files
- HTML report

There are also links to the working directories that get placed in the output
directory. Additionally, a copy of the GTF that was used for expression counts
is placed in the output directory.

[Return to top](#top)
##### PBS Output Files
These files are the `bulk_rnaseq_single_sample.pbs.o...`,
`bulk_rnaseq_single_sample.pbs.e...`, `run_summary_stats.pbs.o...`, and the
`run_summary_stats.pbs.e...` files. They are generated by the scheduler and
contain the text sent to the standard output and standard error channels of the
programs called by the programs of the pipeline. The situation in which you
would most want to view the contents of these files is if your pipeline jobs
are throwing error messages. For now, we do not have to examine them.

[Return to top](#top)
##### Insert Size Metrics
These files are located in the `InsertSizeMetics/` directory, and contain
per-sample insert size summaries. It is only generated if the data are
paired-end. We will not examine these files for the tutorial, but they will
be useful if you plan to submit your data to a database that requires summaries
of the insert sizes, such as NCBI GEO.

[Return to top](#top)
##### Expression Counts
This is one of the main outputs of the `bulk_rnaseq` pipeline. There are two
types of counts that get produced by the pipeline, raw counts, and counts per
million (CPM). CPM expression values are normalized counts divided by the number
of fragments that were sequenced and multiplied by 1,000,000.

The raw counts are stored in the `subread_counts.txt` file. This is the output
from the `featureCounts` program from the `subread` package. In addition to the
raw counts, this file also contains feature lengths. This may be useful if you
would like to use the raw counts in a more sophisticated analysis than the
pipelines implement. In this file, each gene is a row, and samples or gene
descriptors (like length) are columns.

The CPM values are stored in the `cpm_list.txt` file. These counts have been
normalized for library size, then scaled to CPM values. Like the raw counts,
this file has genes as rows and samples as columns. However, the CPM file does
not have gene lengths.

[Return to top](#top)
##### Summary Plots
This is another of the main outputs of the `bulk_rnaseq`. They are located in
the `Plots/` directory. There are three plots that are produced. They are the
distribution of CPM values per-sample, a heatmap of gene expression for the
500 genes with the highest variance among samples, and a multidimensional
scaling (MDS) plot showing sample distances based on gene expression.

An example of the CPM plot is shown below.

![CPM]({{ "/graphics/rnaseq_cmd/cpm_plot.png" | prepend: site.baseurl }})

Note that if you did not supply experimental groups, then the plots will not
be colored. These plots are violin plots, which show the kernel density for each
sample. They are perhaps more informative than barplots, because it is easier to
see the shapes of the distributions.

An example of the heatmap of high variance genes is shown below.

![Heatmap]({{ "/graphics/rnaseq_cmd/high_variance_heatmap.png" | prepend: site.baseurl }})

The blue and red bars that correspond to sample groups will not be present if
you did not supply experimental groups. This plot shows samples as columns and
genes as rows. Samples are clustered based on similarity, with the distance
relationships shown with the dendrogram. Gene expression (CPM) is displayed as
rows, with red colors corresponding to low expression and yellow/white colors
corresponding to high expression. Only the 500 genes with the highest variance
among samples are shown. From the plot it looks like there may be an issue with
separation of the groups. `Spleen.2` and `BoneMarrow.2` look like they might
be swapped labels.

An example of the MDS plot is shown below.

![MDS]({{ "/graphics/rnaseq_cmd/mds_plot.png" | prepend: site.baseurl }})

MDS is a way to identify factors that explain similarities and dissimilarities
among the samples. Samples that are more similar (have lower distance) appear
closer together on the plot. In this case, we can see the samples that appear to
have swapped labels as the single blue label in a group of red, and vice-versa.
We can also see that samples `Spleen.4` and `BoneMarrow.4` are not very tightly
clustered with the other samples.

[Return to top](#top)
##### Differentially Expressed Gene (DEG) List
This file is written to the `DEGs/` directory in the output folder. Use `head`
to look at the first six lines:

```
% head DE_Spleen-BoneMarrow_list.txt
genes   logFC   logCPM  F   PValue  FDR
```

There are no differentially expressed genes!

[Return to top](#top)
##### HTML Report
Finally, there is a summary report generated for the entire run. This is a large
HTML file with embedded graphics and PDF objects. Much of the information in
the HTML report is compiled from the summaries that we just went through above.
Use FileZilla (or your favorite SFTP client) to copy the 
`Bulk_RNAseq_Report.html` file from the specified output directory to your
desktop. Open it in the browser and take a few minutes to scroll through it.

You can view an example of the HTML report [here]({{"/materials/rnaseq_cmd/Bulk_RNAseq_Report.html" | prepend: site.baseurl }}).
Warning: it is large (approximately 3MB)!

[Return to top](#top)
#### <a name="2.3.4"></a> 2.3.4: Fixing Sample Labels
One thing that could be contributing to the fact that we don't have any
differentially expressed genes is that it appears as though there was a mix-up
between a `Spleen` sample and a `BoneMarrow` sample. `Spleen-2` and
`BoneMarrow-2` look like they were swapped, in particular. This is apparent in
the MDS plot and in the clustering heatmap. Let's fix this.

Edit the `Groups.csv` file that you made in **2.3.1** and re-assign labels for
the problematic samples.

```
SampleName,Group
BoneMarrow-1,BoneMarrow
BoneMarrow-2,Spleen
BoneMarrow-3,BoneMarrow
BoneMarrow-4,BoneMarrow
Spleen-1,Spleen
Spleen-2,BoneMarrow
Spleen-3,Spleen
Spleen-4,Spleen
```

Re-run the pipeline with the same command, but be sure to add the `--purge`
option. This tells the pipeline to regenerate alignments.

```bash
% cd /home/msistaff/public/RNAseq_Tutorial/CHURP
% python churp.py bulk_rnaseq \
    -e /scratch.global/YOUR_USER_NAME/RNAseq_Tutorial_Out/Groups.csv \
    -f /home/msistaff/public/RNAseq_Tutorial/Reads \
    -x /home/msistaff/public/RNAseq_Tutorial/Reference/GRCm38_19 \
    -g /home/msistaff/public/RNAseq_Tutorial/Reference/Annotations.gtf.gz \
    -o /scratch.global/YOUR_USER_NAME/RNAseq_Tutorial_Out \
    -d /scratch.global/YOUR_USER_NAME/RNAseq_Tutorial_Work \
    --unstranded \
    --ppn 4 --mem 12000 -w 2 --submit --purge
```

Now, take a look at the plots and the top of the DEG list file. They should
look much more reasonable.

![Swapped CPM]({{ "/graphics/rnaseq_cmd/cpm_plot_swap.png" | prepend: site.baseurl }})

![Swapped Heatmap]({{ "/graphics/rnaseq_cmd/high_variance_heatmap_swap.png" | prepend: site.baseurl }})

![Swapped MDS]({{ "/graphics/rnaseq_cmd/mds_plot_swap.png" | prepend: site.baseurl }})

And when we look at the top of the DEGs list file, we get output:

```
genes              logFC             logCPM           F                PValue               FDR
ENSMUSG00000044220 6.13112504441889  8.08563573490181 338.587778351081 3.9497459543965e-09  1.48998020488011e-06
ENSMUSG00000024810 6.2505506355037   8.7831070255066  315.810066440887 5.57001945749575e-09 1.48998020488011e-06
ENSMUSG00000024659 -4.17943685057531 15.0648102803939 280.891282225036 9.9163938687989e-09  1.76842357326914e-06
ENSMUSG00000024953 -2.09900592323709 13.2553761579828 244.651685142293 1.95184942030757e-08 2.61059859966137e-06
ENSMUSG00000025215 11.7605156086712  8.89647479464905 561.246419555895 4.63615063171065e-08 4.9606811759304e-06
ENSMUSG00000024680 -4.11471280910026 9.50245335028549 183.152399522069 7.97641443190176e-08 7.11230286844573e-06
ENSMUSG00000037071 2.14458600353721  12.5489522373596 148.107571697157 2.21315471124349e-07 1.69148252930752e-05
ENSMUSG00000025083 3.73161698219916  8.30730527741837 137.203730521482 3.18642778575886e-07 2.13092358172623e-05
ENSMUSG00000025060 -1.51128832611889 12.8822670060406 132.426442306639 3.7705700500812e-07  2.24139441865938e-05
```

<div class="warn" markdown="1">

If you do not get any differentially expressed genes in your actual analysis it
does not necessarily mean that there was a sample label issue. This was an
engineered example to illustrate how to read the plots and make inference
about sample quality. The results of a real experiment depend on technical
factors:

- Quality of RNA extraction
- Quality of library preparation
- Depth of sequencing
- Number of replicates

And on biological factors:

- Type of input material (single cells, tissue sample, organ sample, etc.)
- Change in gene expression that is effected by the experimental conditions
- Nested, multi-factorial, or time-series designs

Please see [**Part 6**](#6) for information on more complex designs.

</div>

[Return to top](#top)
### <a name="2.4"></a> 2.4 Data Sources
These data were downloaded from the NCBI Sequence Read Archive (SRA) accession
number SRP164768. Files were renamed to the same format that the UMGC will
deliver to your MSI group. Reads were filtered to contain fragments that map to
mouse chromosome 19, along with approximately 15% reads from other chromosomes.
The reference and annotated gene set are subset versions of the MM10 reference
genome and annotation to only include chromosome 19.

The following perturbations to the subset reads were performed:

- `Spleen-1` was subject to random reductions in base quality
- `BoneMarrow-3` was subject to random reductions in base quality
- `BoneMarrow-4` was subject to random sequence duplication
- `Spleen-2` and `BoneMarrow-2` labels were swapped

We have made attempts to include situations that you may encounter in actual
experiments, such as PCR duplication and low quality reads in your dataset. If
you need consultation on data analysis, or would like to contract the RIS group
at MSI to analyze data for you, please contact the MSI help desk at
<help@msi.umn.edu>.

[Return to top](#top)
<div class="info" markdown="1">

## <a name="3"></a> Part 3: Detailed Description of Pipeline Steps
This section contains a detailed description of all constituent steps of the
`bulk_rnaseq` pipeline:

1. Summarize read quality
2. Clean reads for low-quality bases and adapter contaminants
3. Map reads to genome
4. Count reads within genes
5. Filter counts for genes with low expression
6. Test for differential expression

Steps **1**, **2** and **3** occur in a task array, with one task per sample.
Steps **4**, **5**, and **6** occur in in a single job that processes the output
from each single-sample task.

[Return to top](#top)
### <a name="3.1"></a> 3.1: Summarizing Read Quality
The first part of the pipeline uses `fastqc` to generate summaries of read
length, base quality, base composition, and non-biological sequence
contamination. These reports are written to the working directory for each
sample. Both the HTML report and the text report (in the FastQC-produced ZIP
file) are made available. For more information on how to read the quality
reports generated by FastQC, please refer to our other tutorial that treats
[Illumina data quality control in depth](https://pages.github.umn.edu/MSI-RIS/Tutorials/illumina_galaxy/).

This step is always run. You should view the FastQC reports to see if there are
technical explanations for patterns you observe in your results.

[Return to top](#top)
### <a name="3.2"></a> 3.2: Cleaning Reads
The second part of the pipeline uses Trimmomatic to clean low quality bases,
adapter contamination, and reads that fail length filters from your input
datasets. This step is optional - there are arguments in favor of and against
cleaning reads. To disable read cleaning, simply add the `--no-trim` option to
the `churp.py bulk_rnaseq` command. We enable cleaning by default because it 
s relatively fast and in most cases, will not negatively impact a differential
expression analysis. 

The default options we have set for Trimmomatic are as follows:

1. `ILLUMINACLIP:[adapters]:4:15:7:2:true`
2. `LEADING:3`
3. `TRAILING:3`
4. `SLIDINGWINDOW:4:15`
5. `MINLEN:18`

These correspond to the following operations:

1. Trim contamination from Illumina adapter sequences. Allow up to 4 seed
   mismatches when searching for adapter sequences. Require an alignment score
   of 15 to trim adapters in "palindrome" mode (which corresponds to 25 bases
   of perfect match). Require an alignment score of at least 7 for "simple"
   mode, which corresponds to at least 12 bases of perfect adapter match at
   the end of one read. Require at least 2 bases of adapter sequence to be
   detected for trimming. Do keep both reads after cleaning.
2. Remove bases with a quality score of less than 3 at the start of a read.
3. Remove bases with a quality score of less than 3 at the end of a read.
4. Trim bases with a windowed quality trimming model, removing parts of the read
   until it passes a threshold. We set the window size to 4 bases and require
   that the mean quality of those bases in the window to be at least 15.
5. After all adapter and quality trimming, remove reads that are shorter than
   18 bases.

It is possible to override these options by supplying the
`--trimmomatic-opts=" ... "` option to the `churp.py bulk_rnaseq` call. Pleas
note that any options you supply will **totally override** the defaults that we
set. If you want to only change one of our options, then you must specify
the entire options string with your desired change inserted. Please
also note that if you supply alternative options to Trimmomatic, then the onus
is on you to ensure that they are valid and produce the desired output. We
cannot guarantee that any combination of options outside of our defaults will
work properly for any given dataset. Please consult the Trimmomatic
manual for a full description of available options.

After Trimmomatic cleaning, FastQC is run on the cleaned reads. You can then
compare the raw and the processed reads to determine if you need to adjust any
of the parameters for cleaning.

[Return to top](#top)
### <a name="3.3"></a> 3.3: Mapping Reads
Read mapping is performed with the splice-aware aligner, HISAT2. Note that we
have moved away from TopHat2, which has been deprecated by its authors. It is
possible to supply many options to modify how HISAT2 aligns reads against the
genome. By default, we supply the following options:

1. `--no-mixed`
2. `--new-summary`
3. `-p [ppn]`

These correspond to the following operations:

1. Do not separate pairs of reads when one mate can be mapped but the other
   cannot. By default, HISAT2 separates mates when one cannot find an alignment.
2. Output a summary format that is easier to process with scripts
3. Set the number of threads to the number of processors (can be changed with
   the `--ppn` option to `churp.py bulk_rnaseq`) 
Note that these three options will always be specified, so you do not need to
re-list them when supplying custom options with `--hisat2-opts=" ... "`. Like
with the Trimmomatic custom options, we cannot guarantee that any combination of
options will work appropriately for your dataset. We further cannot validate
any options that are passed; if your option string contains a typo or an illegal
value, it will cause HISAT2 to throw an error and the pipeline will abort.
Please consult the HISAT2 manual for the full list of options that are available
for alignment.

After alignment, the resulting SAM file is sorted by read name (for
fragment-based counting) and converted to BAM.

[Return to top](#top)
### <a name="3.4"></a> 3.4: Counting Reads in Genes
Once the alignments are generated and sorted by read name, they are used as
input for the `featureCounts` program, part of the `subread` package. We count
*fragments* that map to genes to generate expression counts. That is, if the
library is paired-end, then both mates must map to the same gene for it to be
counted as originating from that gene. If the library is single-end, then
individual reads represent sequenced fragments, and reads are counted if they
map to annotated genes.

We also support two types of strandedness for the libraries, unstranded, and
reverse-stranded. By default, the UMGC prepares RNAseq libraries using a
reverse-stranded protocol. Thus, the default option for our `featureCounts` call
is to count reverse-stranded alignments. Additionally, we also set a minimum
mapping quality of 10 to reads that are considered as valid alignments for
counts. If reads are not confidently mapped to a single location, then they are
not used for counts.

[Return to top](#top)
### <a name="3.5"></a> 3.5: Filtering and Differential Expression Testing
The raw counts matrix generated by `featureCounts` is then processed with a
R script that filters the counts matrix, generates summary plots, calculates a
normalized value for gene expression, and tests for differentially expressed
genes. To enable the differential expression testing, you must specify
experimental groups by following the directions in **2.3.1** above.

The filters are applied on a gene-basis. Genes are filtered based on length
and variance in expression across all samples. If the variance in raw counts
is less than `1`, then the gene is removed from the matrix. If the gene is
less than the minimum gene length (tunable with the `--min-gene-length` option,
default of 200bp), then it is removed from the matrix.

The resulting expression counts matrix is then processed with the EdgeR package
for RNAseq analysis. The raw counts are transformed to counts-per-million (CPM),
and written to disk. Distributions of CPM for each gene and each sample are
plotted in violin plots. The 500 genes with the highest variance among the
samples are plotted in a clustered heatmap. Finally, a multidimensional
scaling (MDS) plot is generated for the samples.

Several conditions must be met for differential expression testing to take
place. First, experimental groups must be specified. If there are more than
five groups or fewer than two groups, then we do not perform differential
expression testing as part of the pipelines. This is because the experimental
design is potentially to complex to analyze with a series of pairwise
differential expression analyses. If there are fewer than three replicates per
experimental group, we will also not perform differential expression testing.
In either of these cases, if you would like to analyze your data in a different
way, then you may perform your analyses with the normalized counts matrix or the
raw counts matrix. You may contact the University of Minnesota Informatics
Institute or the Research Informatics Solutions group at MSI for assistance
with data analysis that falls outside of the `CHURP`.

[Return to top](#top)
## <a name="4"></a> Part 4: Getting Your Own Reference Genome Index Files
### <a name="4.1"></a> 4.1: HISAT2 Pre-Built Indices
The developers of HISAT2 maintain a collection of genome indices that are usable
as targets for alignment (`-x` flag in our pipeline). You can find the various
indices that are available on the right panel of the [HISAT2 homepage](https://ccb.jhu.edu/software/hisat2/index.shtml).
Note that version of the genome assembly, version of the annotation, and the
known SNPs and slice sites are important pieces of information to include.

We generally recommend that you use index files that were built with known
SNPs and splice sites. These are the `genome_snp_tran` files that are linked on
the site. The inclusion of known SNPs and splice sites allows you to map reads
accounting for the known structural differences between transcripts and the
genome assembly and known sequence polymorphism between individuals in the
species.

[Return to top](#top)
### <a name="4.2"></a> 4.2: Building Your Own Index
As of this writing (November 2018), the pre-built indices are only available
for human, house mouse, Norway rat, common fruit fly, *C. elegans*, and baker's
yeast. If you are analyzing data from a species that does not have a pre-built
index, then you will have to produce your own.


First, download a reference genome assembly and a gene annotation. A good source
of reference genome assemblies is [Ensembl](https://ensembl.org). Make sure that
you get a FASTA file for the genome assembly and a GTF file for the gene
annotations. HISAT2 does not take gzipped input files for index building, so
they must be unzipped. Additionally, if your study species has such information
available, download a dbSNP build that includes polymorphisms discovered in your
species of interest. You can download them from the [UCSC Genome Browser](http://hgdownload.cse.ucsc.edu/downloads.html).
If you do not have a dbSNP dump for your species of interest, then you can use
a VCF.

<div class="warn" markdown="1">

Note that not all SNPs in public databases (or even those derived from your own
high-throughput sequencing analysis) are "real." Many are the result of poor
alignment in regions of the genome that are difficult to assemble, or
alignment errors involving short insertions/deletions. This shouldn't affect
the index building procedure, but it is important to be aware of the error-prone
nature of any "-omics" dataset.

</div>

Next, use auxiliary scripts that are bundled with HISAT2 to extract the splice
sites and SNPs into a format that the index builder will read. These become
available once you load the `hisat2` module on MSI systems. The names of the
files are `hisat2_extract_snps_haplotypes_UCSC.py` for dbSNP dumps,
`hisat2_extract_snps_haplotypes_VCF.py` for VCFs, and 
`hisat2_extract_splice_sites.py` for GTFs. They write data to standard out,
so be sure to redirect with the `>` character on the shell. The scripts are
reasonably light, and can be run in an interactive HPC session. Note that at the
time of this writing (November 2018, HISAT2 version 2.1.0), the auxiliary
scripts require `python2`, and will not work with `python3`.

Once you have SNPs and splice sites in HISAT2 input files, write the necessary
commands for index building into a shell script following the
[build instructions](https://ccb.jhu.edu/software/hisat2/manual.shtml#the-hisat2-build-indexer).
This should be formatted as a PBS job script and sent to the scheduler.

<div class="warn" markdown="1">

Please note that building index files for HISAT2 are memory intensive tasks, so
you will likely need to use the `ram256g` or `ram1t` queues to run your jobs.
For limits on these queues, please refer to the [MSI queues page](https://www.msi.umn.edu/queues).

For example, a build of the mm10 house mouse genome with known splice sites
required approximately 350GB of memory.

</div>

[Return to top](#top)
## <a name="5"></a> Part 5: Pipeline File Formats
The `bulk_rnaseq` pipeline of `CHURP` generates up to three
pipeline-specific file formats. Technically, these files are not proprietary
in any way, and can be used by third-party applications and custom user scripts.
However, we cannot support the diversity of ways that people may interact with
the internal files of `CHURP`.

[Return to top](#top)
### <a name="5.1"></a> 5.1: Experimental Groups Template
This is a comma-separated values (CSV) file that contains placeholders for the
experimental for the samples in question. By default, it contains only two
fields, `SampleName` and `Group`. The `SampleName` field contains the inferred
name of the sample, derived from the filename of the FASTQ file. The `Group`
column is auto-populated with a `NULL` value. We do not make any attempt to
automatically assign samples to groups based on patterns in the filenames.

[Return to top](#top)
### <a name="5.2"></a> 5.2: Pipeline.sh Script
This is a standard bash script that contains paths to the samplesheet and the
PBS job scripts. The template for it is shown below:

```bash
#!/bin/bash
# Generated by CHURP version 0.0.1
# Generated at 2018-12-04 15:31:08
set -e
set -u
set -o pipefail
user_name="$(id -u -n)"
user_email="${user_name}@umn.edu"
OUTDIR="/scratch.global/konox006/RNAseq_Tutorial_Out/"
WORKDIR="/scratch.global/konox006/RNAseq_Tutorial_Work/"
DE_SCRIPT="/panfs/roc/groups/14/msistaff/public/RNAseq_Tutorial/CHURP/R_Scripts/summarize_bulk_rnaseq.R"
REPORT_SCRIPT="/panfs/roc/groups/14/msistaff/public/RNAseq_Tutorial/CHURP/R_Scripts/bulk_rnaseq_report.Rmd"
SAMPLESHEET="/scratch.global/konox006/RNAseq_Tutorial_Out/2018-12-04.konox006.bulk_rnaseq.samplesheet.txt"
PURGE="true"
PIPE_SCRIPT="$( cd "$( dirname "${BASH_SOURCE[0]}" )" >/dev/null && pwd )/$(basename $0)"
single_id=$(qsub -q mesabi <options> [...]bulk_rnaseq_single_sample.pbs || exit 1)
summary_id=$(qsub -q mesabi <options> [...]run_summary_stats.pbs || exit 1)
echo "Output and logs will be written to ${OUTDIR}"
echo "Emails will be sent to ${user_email}"
echo "Single samples job array ID: ${single_id}"
echo "Summary job ID: ${summary_id}"
```
Note that long paths and option strings have been truncated for readability.
The files that you generate will have full paths and options lists in them.

Running this script will submit the entire pipeline to the scheduler. The only
path that you will have to check is the `SAMPLESHEET=` declaration. This should
be the location of the samplesheet that was generated by `churp.py`.

[Return to top](#top)
### <a name="5.3"></a> 5.3 Samplesheet 
The samplesheet contains the information necessary to process each sample, such
as the paths to the reads, the reference genome for alignment, and trimming
options that you have specified. The file is very "wide" (has long lines), so
we will not reproduce it verbatim.

The format is a series of pipe-delimited (`|`) fields. For version `0.0.1` of
the `CHURP`, the fields are in the following order:

1. Sample name
2. Experimental group
3. R1 path
4. R2 path
5. Output directory
6. Working directory
7. Trim ("yes" or "no")
8. Remove duplicates ("yes" or "no")
9. Trimmomatic options
10. HISAT2 reference genome index
11. HISAT2 options
12. Unstranded ("yes" or "no")
13. GTF path

The final lines contain meta-information for the samplesheet. It has the date
the sheet was generated and the version of the pipelines that generated the
script.

It is important that the samplesheet starts directly with sample information.
There is **no header line**. You can put any free-form information between the
block of sample data and the comment lines. The PBS worker scripts parse the
sample data block and the final line in the file, so they must remain intact
for the scripts to work.

[Return to top](#top)
## <a name="6"></a> Part 6: More Complex Analyses
This section will cover how to prepare for more complicated analyses than we
support in `CHURP`. Examples of these types of analyses are
experiments with nested designs, multi-factorial comparisons, or time series
analyses. The example data set used in this tutorial is actually part of a
more complex design that we simplified for the purposes of the tutorial.

We cannot provide catch-all instructions for analyzing more complicated
designs because each experiment will need its own procedure for analysis. You
can consult a statistician or the RIS group at MSI for assistance. We can,
however, provide some mechanisms for helping you analyze more complex datasets
by automating some of the more tedious parts of the setup.

[Return to top](#top)
### <a name="6.1"></a> 6.1: `group_template` Columns
By default, the `group_template` subcommand only makes a `SampleName` and a
`Group` column. You can specify the `-e` option to add other columns to the
groups CSV file. The argument given to the `-e` option is the name of the
column, and the option can be specified multiple times.

For example, a more complete set of information for the tutorial samples is
shown below. Note that this reflects the **true** data set, and not the labels
that were swapped for instructional purposes.

| Name         | Genotype | Tissue      | Time |
|--------------|----------|-------------|------|
| BoneMarrow-1 | Elf4-/-  | Bone marrow | 2    |
| BoneMarrow-2 | Elf4-/-  | Bone marrow | 0    |
| BoneMarrow-3 | WT       | Bone marrow | 2    |
| BoneMarrow-4 | WT       | Bone marrow | 0    |
| Spleen-1     | Elf4-/-  | Spleen      | 2    |
| Spleen-2     | Elf4-/-  | Spleen      | 0    |
| Spleen-3     | WT       | Spleen      | 2    |
| Spleen-4     | WT       | Spleen      | 0    |

In our example, we used the tissue type as the group, but we can specify
additional columns by modifying our original call:

```bash
% python churp.py group_template bulk_rnaseq \
    -f /home/msistaff/public/RNAseq_Tutorial/Reads \
    -e Genotype \
    -e Tissue \
    -e Time \
    -o /scratch.global/YOUR_USER_NAME/RNAseq_Tutorial_Out/Groups.csv
```

The resulting CSV file will look like this:

```
SampleName,Group,Genotype,Tissue,Time
BoneMarrow-1,NULL,NULL,NULL,NULL
BoneMarrow-2,NULL,NULL,NULL,NULL
BoneMarrow-3,NULL,NULL,NULL,NULL
BoneMarrow-4,NULL,NULL,NULL,NULL
Spleen-1,NULL,NULL,NULL,NULL
Spleen-2,NULL,NULL,NULL,NULL
Spleen-3,NULL,NULL,NULL,NULL
Spleen-4,NULL,NULL,NULL,NULL
```

Note that `SampleName` and `Group` are always present: these are required for
the `edgeR` testing that we do. You can then fill in the appropriate values
for the experimental variables.

You can leave the `Group` column as all `NULL` values when running the
pipelines. If you do this, then the pipeline will stop after generating raw and
normalized counts. You may then use the counts matrix and the groups CSV file
to run a statistical analysis in the package of your choice.

With more detailed metadata, it is possible to model the effects of factors
like the technician who extracted the RNA, the batch in which a library was
extracted, the enclosure of the individual in the experiment, the protocol that
was used for extraction and storage, and so on. Always keep good notes on these
factors, because they may end up explaining a large portion of the variance in
observed gene expression.

[Return to top](#top)
### <a name="6.2"></a> 6.2 Links to Analysis Guides
Various packages exist to analyze expression data. We have chosen to use `edgeR`
in our pipeline, but the counts matrix and group CSV are compatible with many
others. In fact, one of the main use cases for `CHURP bulk_rnaseq` is to
automate the generation of an expression counts matrix and metadata CSV so that
gene expression can be analyzed in complex designs with specialized packages.

[DESeq2](https://bioconductor.org/packages/release/bioc/html/DESeq2.html) is
another popular package for analyzing RNAseq data. It uses the same negative
binomial model of read counts as `edgeR`.

The [edgeR Manual](https://www.bioconductor.org/packages/release/bioc/vignettes/edgeR/inst/doc/edgeRUsersGuide.pdf)
also contains useful information for analyzing more complex designs.

For time-series analysis, [DyNB](https://research.cs.aalto.fi/csb/software/dynb/)
uses a Gaussian process to model temporal correlations among gene expression
profiles in combination with the negative binomial model for expression counts.
Additionally, [GPClust](https://github.com/SheffieldML/GPclust) can be used to
identify modules of genes with similar expression profiles over time.

[Return to top](#top)
## <a name="7"></a> Part 7: Recommendations
### <a name="7.1"></a> 7.1: General Considerations for Experimental Design
In reference to the warnings given above, it is important to design powerful
experiments and keep detailed notes on your samples. Your experiments should
have a clear control for each experimental treatment. Your null hypothesis
should be clearly defined, and you should have an expectation for the difference
between your experimental conditions. Your samples should be randomized across
technical variables (like extraction batch) whenever possible. Strong
informatics skills and a clever analytical workflow cannot make meaningful
results from an underpowered experiment. However, empirical scientific research
is about balancing insight and practical limitations. A good analytical workflow
can help make sure that you get the most value out of your data.

[Return to top](#top)
### <a name="7.2"></a> 7.2: How Much Sequence Data to Collect
For differential gene expression analyses in most animals or plants, we
recommend a minimum of 20 million reads per sample. For simpler eukaryotes
(e.g., nematodes or yeast), 10 million reads per sample may suffice. You should
have at least three biological replicates (i.e., samples from three different
individuals or clonal pools) per experimental condition, if possible. There are
cases where three replicates is excessive, such as high-resolution time series
analyses, so use your best judgment.

The UMGC recommends 50bp paired-end sequencing technology for differential
gene expression analysis. For isoform-level expression or alternative splicing
analysis, they recommend at least 125bp paired-end technology. It is possible
to use single-end reads for differential gene expression, but it is not a widely
requested technology, and it may take longer to move through the queue.

The ENCODE project maintains [guidelines for RNAseq experiments](https://www.encodeproject.org/documents/cede0cbe-d324-4ce7-ace4-f0c3eddf5972/@@download/attachment/ENCODE%20Best%20Practices%20for%20RNA_v2.pdf) that may be helpful.
You may also refer to [this page from Genohub](https://genohub.com/recommended-sequencing-coverage-by-application/)
that lists recommendations for coverage for a variety of applications.

If you are assembling a transcriptome, you should plan to collect at least 40
to 50 million reads per sample. Collecting reads from multiple tissues,
developmental time points, or experimental treatments is beneficial because
many more transcripts are potentially represented than in any individual
library.

You may view information about the HiSeq2500, NextSeq, and NovaSeq that the UMGC
operates at the following links:

- [HiSeq2500](http://genomics.umn.edu/nextgen-hiseq-high.php)
- [NextSeq](http://genomics.umn.edu/nextgen-nextseq.php)
- [NovaSeq](http://genomics.umn.edu/nextgen-novaseq.php)

[Return to top](#top)
#### <a name="7.2.1"></a> 7.2.1: UMGC Pricing
Pricing depends on the instrument that you are requesting, the output mode
that is being used, and whether your appointment is internal or external to the
University of Minnesota. The pricing guides for the HiSeq2500, NextSeq, and
NovaSeq at the UMGC are linked below:

- [HiSeq2500](http://genomics.umn.edu/nextgen-hiseq-high.php), click the "Pricing" tab.
- [NextSeq and NovaSeq Internal](http://genomics.umn.edu/downloads/NovaSeq_NextSeq_Internal_Pricing_032118_update.pdf)
- [NextSeq and NovaSeq External](http://genomics.umn.edu/downloads/NovaSeq_NextSeq_External_Pricing_032118_update.pdf)

For a quote, you can contact the next-gen sequencing team UMGC by emailing
<next-gen@umn.edu> with your desired read quantity, read length, and
single/paired technology. They will prepare a quote that will be good for three
months from the preparation date.

For a full description of UMGC services and pricing, please see their
[catalogue]({{"/materials/rnaseq_cmd/UMGC_2015-16_Catalog_Internal.pdf" | prepend: site.baseurl }}).
Note that despite what the first page says, the prices are still current,
as of November 2018. **The UMGC is updating their pricing schedule, however,
so prices are subject to change in the next six months.**

[Return to top](#top)
#### <a name="7.2.2"></a> 7.2.2: Expected Turnaround Time
The standard turnaround time for RNAseq projects at the UMGC is 6-8 weeks,
assuming the libraries pass QC and there are no complications with the reagents
or the instruments. For faster turnaround, you may request to use the NextSeq,
which is a single lane, rather than a shared cell like with the other
instruments.

[Return to top](#top)
### <a name="7.3"></a> 7.3: Analytical Workflow Considerations
There are a few additional concerns for RNAseq experiments:

1. Be careful with data from public sources
2. Use "best in class" software
  - HISAT2, STAR
  - Kallisto (but only for specialized applications)
  - EdgeR or DESeq2 for differential expression analysis
3. Be careful with tools built for microarrays

In relation to point **1** above, the technical variables for the data set used
in this tutorial were not available. The strain, age at sacrifice, and the
experimental conditions are available in the SRA, but RNA quality summaries and
sample handling information is not available. Collaborative work between some
members of the RIS group and UMGC Funded Services group has shown that technical
variables such as sample storage can drive major patterns of differentiation
of gene expression profiles. Such metadata are not often deposited with public
data, so be cautious when analyzing data from the SRA with no technical
metadata.

In regard to point **2**, most RNAseq alignment should be done with HISAT2 or
STAR. STAR requires that you build a special index for each read length that
you wish to align, so we cannot provide general instructions here. Kallisto can
be used for transcript quantification, but is not a general purpose alignment
tool for expression analysis - you should only use Kallisto if your reads and
reference genome have **very few** expected sequence differences. Most studies
fall outside of this case, so we do not recommend it for general RNAseq
analyses. Several packages and statistical models exist for identifying
differentially expressed genes, but we recommend EdgeR or DESeq2 because they
implement robust models and have well-documented features.

For point **3**, the nature of the data from microarrays is fundamentally
different from that of RNAseq. Microarrays use hybridization, which is a
fluorescence-based signal, rather than a counts-based signal. Be sure that the
package you are using can handle RNAseq data before applying it.

[Return to top](#top)
## <a name="8"></a> Part 8: Other RNASeq Applications
Finally, differential expression analysis is only a small set of what is
possible with RNAseq data. Other types of analysis include co-expression
analysis, transcriptome assembly and isoform discovery, and variant discovery
in the absence of a reference genome.

[Return to top](#top)
### <a name="8.1"></a> 8.1 Coexpression
The gist of a coexpression analysis is that it identifies modules of genes that
have similar expression profiles. That is, genes that are either expressed under
the same conditions or are expressed in the same tissues. A popular package for
coexpression is **W**eighted **G**ene **C**orrelation **N**etwork **A**nalysis
(WGCNA). This package will identify groups of genes that have high correlations
within groups, and low correlations between groups, which can identify groups
of genes that respond to experimental conditions or developmental time points.

If you choose to use WGCNA, be sure to input the normalized counts, rather
than the raw counts.

[Link to WGCNA manual](https://horvath.genetics.ucla.edu/html/CoexpressionNetwork/Rpackages/WGCNA/)

[Return to top](#top)
### <a name="8.2"></a> 8.2 Transcriptome Assembly
You can use RNAseq to assemble putative transcript sequences from your organism
of choice. A popular tool for assembling transcriptome sequences from short
reads is Trinity. Trinity also has some capability of identifying isoforms of
the same gene, but short reads inherently have little information about which
isoform was sequenced.

If you choose to use Trinity, be sure to sequence multiple libraries from the
same organism. Combining libraries from multiple tissues, developmental
time points, or experimental conditions is also beneficial, because you can
capture as broad a transcriptome as possible. Trinity also has very high runtime
and memory requirements. Filtering reads from contaminant organisms or
ribosomal sequences may help save on computational time.

[Link to Trinity manual](https://github.com/trinityrnaseq/trinityrnaseq/wiki)

[Return to top](#top)
### <a name="8.3"></a> 8.3 Variant Discovery
Despite the increasing accessibility of genomics technologies, the practical
cost of genome sequencing and assembly is still very high. In the cases where
a reference genome assembly is not available for an organism, a reference
transcriptome may suffice. The "gene space" of many eukaryotic organisms does
not vary on the same orders of magnitude as the whole genome. In many cases,
transcribed regions of the genome are the regions of interest and direct
interpretability because variants discovered in transcripts have higher
potential to have direct effects on peptide sequences, and thus organismal
development, physiology, and behavior. Alternately, RNAseq may be a more
affordable way to sequence a broad sample of an organism, if sequence capture
is not feasible.

If you are discovering variants with only a reference transcriptome, then you
can follow very similar approaches to variant discovery in DNAseq techniques.
If you are using a reference genome, then you must account for alignments that
span intron-exon boundaries. 

The Broad Institute maintains a guide on the best practices for variant
discovery in RNAseq [here](https://gatkforums.broadinstitute.org/gatk/discussion/4067/best-practices-for-variant-discovery-in-rnaseq).

</div>

[Return to top](#top)
## Part 9: Feedback
This tutorial document was prepared by Thomas Kono, in the RIS group at MSI.
Please send feedback and comments to konox006 [at] umn.edu. You may also send
tutorial delivery feedback to that address.

[Return to top](#top)
