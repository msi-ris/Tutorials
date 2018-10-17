---
layout: default
title: RNASeq Analysis With the Command Line
permalink: /rnaseq_cmd/
exclude: false
updated: 2018-10-16
delivered: NA
---

# {{page.title}}
*Last Updated: {{page.updated}}*

*Last Delivered: {{page.delivered}}*


## Part 0: Introduction
### 0.1: Goals
- Learn some fundamental bash commands for interacting with MSI systems
- Log in to MSI systems and access a high-performance compute node
- Become familiar with several core file formats of genomics
- Learn the basics of RNASeq data analysis, including read mapping, expression
  counts, and testing for differential expression
- Learn how to use the UMII-RIS pipelines to perform a differential expression
  analysis with RNAseq

### 0.2: Required Software
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

### 0.3: UNIX (and Linux) Basics
MSI systems run GNU/Linux, which is a functional equivalent of commercial UNIX
(like Apple's OSX). The way that you will interact with Linux is very similar to
the way you would interact with a traditional UNIX system - through the command
line.

![Terminal]({{ "/graphics/rnaseq_cmd/terminal.png" | prepend: site.baseurl }})

Be warned, though, because the GNU/Linux versions of the programs behave
very similarly to the UNIX versions, but they are not identical. Always check
the help or manual pages if you experience some unexpected behavior from one
of the programs that you are running.

#### 0.3.1: Common Commands
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

#### 0.3.2: Other Resources
There are plenty of resources available for learning how to interact with the
computer through the command line. For technical reference, the default
command line program on MSI, or shell, is called `bash`. Here are a few other
links for `bash` resources:

- [UCR Linux Basics](http://hpcc.ucr.edu/manuals_linux-basics_intro)
- [Advaned Bash Scripting Guide from The Linux Documentation Project](https://www.tldp.org/LDP/abs/html/)

### 0.4: Accessing MSI
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
### 1.1: Why RNASeq
- expression
- variant discovery without reference genome
- reduced representation, genes are often the most interpretable pieces of a genome

### 1.2: Common Genomics File Formats
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

### 1.3: Overall Workflow
Here is a schematic of the workflow that we will follow for this tutorial. The
pieces of data are shown in rectangles, and the software are shown in rounded
bubbles with dashed borders. The final output is shown in red.

![Workflow]({{ "/graphics/rnaseq_cmd/workflow.png" | prepend: site.baseurl }})

The steps of the workflow are as follows:

1. Summarize read quality
2. Clean reads for low-quality bases and adapter contaminants
3. Map reads to genome
4. Count reads within genes
5. Filter counts for genes with low epxression
6. Test for differential expression

## Part 2: Running an Analysis With `gopher-pipelines`
The workflow steps outlined in the previous section are implemented in a package
developed and maintained by the University of Minnesota Informatics Institute
and the Research Informatics Solutions groups. This software package is
available on MSI systems as a software module.

You may also download the latest version of the code from the UMN GitHub
instance: https://github.umn.edu/MSI-RIS/gopher-pipeline-refactor. This will
give you greater flexibility, because you will be able to modify the underlying
Python and shell scripts. However, we offer limited support for workflows that
use modified versions of our pipelines.

### 2.1: Prepare to Run `gopher-pipelines`
First, connect to a head node on Mesabi by typing `ssh mesabi` at the login
prompt. If you do no have ssh keys set up, you will have to enter your password
again. It is the same as your X.500 password. Note the `@login` part of the
prompt changes to reflect that you have started a shell on the Mesabi head
node.

![Mesabi]({{ "/graphics/rnaseq_cmd/mesabi.png" | prepend: site.baseurl }})

From here on, the commands and their output will be shown in text, rather than
in screenshots of terminal emulators.

Next, load the pipeline module by typing `module load gopher-pipelines-refactor`
at the command prompt. You can now execute the main pipeline control script,
called `gopher-pipelines.py`.

```
konox006@ln0004 [~] % module load gopher-pipelines-refactor
konox006@ln0004 [~] % gopher-pipelines.py 
Usage: gopher-pipelines.py <subcommand> <options>

where <subcommand> is the name of the pipeline that is to be run. The specified
<options> will be applied to the operations in the pipeline. Each pipeline has
its own set of options that must be specified. To see a full listing of each
available option for a given pipeline, pass the '--help' option. Alternately,
online help is maintained at the GitHub repository.

Currently, the following subcommands are supported:
    - bulk_rnaseq

For issues, contact help@msi.umn.edu.
Version: 0.0
2018-07-24

```

If you get a message similar to the one above, then you are ready to run the
pipelines.

### 2.2: Input Data for the Pipelines
The pipelines requires that the following pieces of data be available:

- Directory of FASTQ files from the UMGC
- Reference genome indexed with HISAT2
- Gene annotations in GTF format

For this tutorial, we have provided these data. The FASTQ files are located at
`/panfs/roc/scratch/data_release/umgc/hiseq/run_id/Example_Project_001`. The
indexed reference genome is located at
`/panfs/roc/scratch/example_analysis/genome/Reference`. The accompanying GTF
gene annotations are located at
`/panfs/roc/scratch/example_analysis/genome/annotations.gtf.gz`.

When you run your own analysis, you may need to generate your own HISAT2 index
for your reference genome. We will provide some brief instructions in a later
section of this tutorial document, but the best source of instructions is from
the HISAT2 manual.

### 2.3: Running the Pipeline
The subcommand that we will be using in this tutorial is the `bulk_rnaseq`
subcommand. Provide it as an argument after the `gopher-pipelines.py` command.
The default help message tells you the bare minimum arguments that are required
to analyze the data. To see a full help message, add the `-h` option after the
`bulk_rnaseq` subcommand. For more detail on the options that are available
for the `bulk_rnaseq` pipeline, you can access the latest version of the manual
at the [GitHub Wiki page](https://github.umn.edu/MSI-RIS/gopher-pipeline-refactor/wiki/bulk_rnaseq).

```
konox006@ln0004 [~] % gopher-pipelines.py bulk_rnaseq
----------
ERROR


You did not specify sufficient options to run the bulk_rnaseq subcommand of
gopher-pipelines. You must specify a FASTQ directory (-f). Additionally, you
must either specify a path to a HISAT2 index (-x) and GTF (-g), or an organism
name (-r). If you are building a group template file, you need only specify a
FASTQ directory. Please fix your command line and re-run.

```

As the help message shows, you must specify a FASTQ directory, a HISAT2 index,
and a GTF file. We will eventually use these files to run an analysis.

#### 2.3.1: Generating Experimental Groups
For differential expression testing, we have to generate a file that describes
which samples are part of which experimental group. We do this with the
`group_template` subcommand. This particular subcommand takes an additional
argument. In this case, we will supply `bulk_rnaseq` because we are generating
a groups file for the `bulk_rnaseq` pipeline. We supply the FASTQ directory as
the argument:

```
konox006@ln0004 [~] % gopher-pipelines.py group_template bulk_rnaseq \
    -f /panfs/roc/scratch/data_release/umgc/hiseq/run_id/Example_Project_001/
----------
SUCCESS

Template file: /panfs/roc/ [ ... ] 8-10-16.konox006.bulk_rnaseq.groups.csv

All sample groups and any additional columns have specified have been filled
with NULL. Please edit the file and write in the correct values for your
dataset. Samples with the same "Group" label will be treated as replicates
in the analysis. Samples with a "Group" value of NULL will not be used in
downstream differential expression analysis. When you have edited the file to
your liking, supply its path to the "bulk_rnaseq" pipeline with the -e
option to enable group testing.

```

Note that the path to the template file has been truncated in this webpage for
readability reasons. When you run the script in the terminal, you will see the
full path printed.

The template file must then be edited to contain the assignments to each group.
By default, all samples are listed as being part of a group labeled `NULL`. The
default file is shown below:

```
SampleName,Group
GrpA-0-07,NULL
GrpA-0-08,NULL
GrpA-0-09,NULL
GrpA-0-15,NULL
GrpB-0-06,NULL
GrpB-0-07,NULL
GrpB-0-13,NULL
GrpB-0-14,NULL
```

Use the `nano` text editor (or your favorite terminal editor program) to assign
the samples to their groups. In this case, the first four samples are part of
group `A` and the final four samples are part of group `B`.

```
SampleName,Group
GrpA-0-07,A
GrpA-0-08,A
GrpA-0-09,A
GrpA-0-15,A
GrpB-0-06,B
GrpB-0-07,B
GrpB-0-13,B
GrpB-0-14,B
```

This is a manual step because the researcher must made a decision as to which
groups should be compared for differential expression testing.

#### 2.3.2: Submitting the Pipeline Jobs
Now that we have assigned the samples to groups, we will run the `bulk_rnaseq`
pipeline, supplying the file that contains the group assignments. This is done
by supplying the `-e` option in addition to the `-f`, `-x`, and `-g` options. We
also supply the `--submit` option to automatically send the jobs to the queue.
If you omit the `--submit` option, then you will have a chance to edit the
pipeline script and the samplesheet. The formats for these files will be
described later in the document.

```
konox006@ln0004 [~] % gopher-pipelines.py bulk_rnaseq \
    -e /panfs/roc/scratch/konox006/2018-10-16.bulk_rnaseq/2018-10-16.konox006.bulk_rnaseq.groups.csv \
    -f /panfs/roc/scratch/data_release/umgc/hiseq/run_id/Example_Project_001/ \
    -x /panfs/roc/scratch/example_analysis/genome/Reference \
    -g /panfs/roc/scratch/example_analysis/genome/annotations.gtf.gz \
    --submit
----------
SUCCESS

Your pipeline has been submitted. For reference, the pipeline script and the
samplesheet are given at the paths below:

Pipeline script: /panfs/roc/scratch/ [ ... ] .bulk_rnaseq.pipeline.sh
Samplesheet: /panfs/roc/scratch/kono [ ... ] k_rnaseq.samplesheet.txt

Below is the output from qsub.
Qsub stdout:
Output and logs will be written to /panfs/roc/scratch/konox006/2018-10-16.bulk_rnaseq
Emails will be sent to konox006@umn.edu
Single samples job array ID: 8635654[].mesabim3.msi.umn.edu
Summary job ID: 8635655.mesabim3.msi.umn.edu

Qsub stderr:
```

The message that prints tells you the IDs of the submitted jobs and the
address to which the email notifications will be sent. For your reference, the
pipeline script and samplesheet paths are also printed. If you would like to
reproduce the analysis, you must keep both the pipeline script and the
samplesheet.

Note that the paths to the pipeline script and the samplesheet have been
truncated in this webpage for readability. When you run the pipeline in the
terminal, you will see the full path printed.

By default these files are stored in scratch, which is cleaned during monthly
maintenance. Be sure to save the files from your analysis before the
maintenance cycle!

You can verify that the jobs are submitted by issuing the `qstat -t` command.
The `-t` expands the job array that we use to process samples in batches.

```
konox006@ln0004 [~] % qstat -t
Job ID                    Name             User            Time Use S Queue
------------------------- ---------------- --------------- -------- - -----
8635654[1].mesabim3.msi.u ..._sample.pbs-1 konox006               0 Q small
8635654[2].mesabim3.msi.u ..._sample.pbs-2 konox006               0 Q small
8635654[3].mesabim3.msi.u ..._sample.pbs-3 konox006               0 Q small
8635654[4].mesabim3.msi.u ..._sample.pbs-4 konox006               0 Q small
8635654[5].mesabim3.msi.u ..._sample.pbs-5 konox006               0 Q small
8635654[6].mesabim3.msi.u ..._sample.pbs-6 konox006               0 Q small
8635654[7].mesabim3.msi.u ..._sample.pbs-7 konox006               0 Q small
8635654[8].mesabim3.msi.u ..._sample.pbs-8 konox006               0 Q small
8635655.mesabim3.msi.umn. ...ary_stats.pbs konox006               0 H small
```

The `Q` means that the job is queued. When the jobs begins to run, it will turn
to `R`. When it finishes, it will become `C`. The `H` means that the job is
held - we set this job to wait until the previous jobs complete successfully
before marking it as eligible to run. The `H` will eventually become `Q`, and
then `R` and `C`. After a while, the job will be cleared from the list.

#### 2.3.3: Viewing Results
Once all of the jobs have finished (all are listed with `C` status, or are
no longer in the output of `qstat -t`), you will be able to view the results.
The output is in the directory that was given as the output directory in the
pipeline output message. For our example, this is
`/panfs/roc/scratch/konox006/2018-10-16.bulk_rnaseq`.

Navigate to the output directory and list the contents

```
konox006@ln0004 [~] % cd /panfs/roc/scratch/konox006/2018-10-16.bulk_rnaseq
konox006@ln0004 [/panfs/roc/scratch/konox006/2018-10-16.bulk_rnaseq] % ls -lF
total 11M
-rw-rw---- 1 konox006 msistaff  137 Oct 16 14:25 2018-10-16.konox006.bulk_rnaseq.groups.csv
-rw-rw---- 1 konox006 msistaff 1.3K Oct 16 15:14 2018-10-16.konox006.bulk_rnaseq.pipeline.sh
-rw-rw---- 1 konox006 msistaff 4.0K Oct 16 15:14 2018-10-16.konox006.bulk_rnaseq.samplesheet.txt
lrwxrwxrwx 1 konox006 msistaff   66 Oct 16 15:07 allsamples_work_directory ->
/panfs/roc/scratch/konox006/2018-10-16.bulk_rnaseq.work/allsamples/
-rw-rw---- 1 konox006 msistaff 8.5M Oct 16 15:07 annotations.gtf.gz
-rw-rw---- 1 konox006 msistaff 2.7K Oct 16 15:06 bulk_rnaseq_single_sample.pbs.e8635654-1
-rw-rw---- 1 konox006 msistaff 2.8K Oct 16 15:06 bulk_rnaseq_single_sample.pbs.e8635654-2
-rw-rw---- 1 konox006 msistaff 2.8K Oct 16 15:06 bulk_rnaseq_single_sample.pbs.e8635654-3
-rw-rw---- 1 konox006 msistaff 2.8K Oct 16 15:06 bulk_rnaseq_single_sample.pbs.e8635654-4
-rw-rw---- 1 konox006 msistaff 2.8K Oct 16 15:06 bulk_rnaseq_single_sample.pbs.e8635654-5
-rw-rw---- 1 konox006 msistaff 2.8K Oct 16 15:06 bulk_rnaseq_single_sample.pbs.e8635654-6
-rw-rw---- 1 konox006 msistaff 2.8K Oct 16 15:06 bulk_rnaseq_single_sample.pbs.e8635654-7
-rw-rw---- 1 konox006 msistaff 2.8K Oct 16 15:06 bulk_rnaseq_single_sample.pbs.e8635654-8
-rw-rw---- 1 konox006 msistaff   88 Oct 16 15:06 bulk_rnaseq_single_sample.pbs.o8635654-1
-rw-rw---- 1 konox006 msistaff   89 Oct 16 15:06 bulk_rnaseq_single_sample.pbs.o8635654-2
-rw-rw---- 1 konox006 msistaff   89 Oct 16 15:06 bulk_rnaseq_single_sample.pbs.o8635654-3
-rw-rw---- 1 konox006 msistaff   89 Oct 16 15:06 bulk_rnaseq_single_sample.pbs.o8635654-4
-rw-rw---- 1 konox006 msistaff   90 Oct 16 15:06 bulk_rnaseq_single_sample.pbs.o8635654-5
-rw-rw---- 1 konox006 msistaff   89 Oct 16 15:06 bulk_rnaseq_single_sample.pbs.o8635654-6
-rw-rw---- 1 konox006 msistaff   89 Oct 16 15:06 bulk_rnaseq_single_sample.pbs.o8635654-7
-rw-rw---- 1 konox006 msistaff   89 Oct 16 15:06 bulk_rnaseq_single_sample.pbs.o8635654-8
drwxrwx--- 2 konox006 msistaff 4.0K Oct 16 15:07 Counts/
drwxrwx--- 2 konox006 msistaff 4.0K Oct 16 15:07 DEGs/
drwxrwx--- 2 konox006 msistaff 4.0K Oct 16 15:07 InsertSizeMetrics/
drwxrwx--- 2 konox006 msistaff 4.0K Oct 16 15:07 Logs/
drwxrwx--- 2 konox006 msistaff 4.0K Oct 16 15:07 Plots/
-rw-rw---- 1 konox006 msistaff 8.9K Oct 16 15:07 run_summary_stats.pbs.e8635655
-rw-rw---- 1 konox006 msistaff    0 Oct 16 15:07 run_summary_stats.pbs.o8635655
lrwxrwxrwx 1 konox006 msistaff   69 Oct 16 15:07 singlesamples_work_directory ->
/panfs/roc/scratch/konox006/2018-10-16.bulk_rnaseq.work/singlesamples/
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

There are also links to the working directories that get placed in the output
directory. Additionally, a copy of the GTF that was used for expression counts
is placed in the output directory.

##### PBS Output Files
These files are the `bulk_rnaseq_single_sample.pbs.o...`,
`bulk_rnaseq_single_sample.pbs.e...`, `run_summary_stats.pbs.o...`, and the
`run_summary_stats.pbs.e...` files. They are generated by the scheduler and
contain the text sent to the standard output and standard error channels of the
programs called by the programs of the pipeline. The situation in which you
would most want to view the contents of these files is if your pipeline jobs
are throwing error messages. For now, we do not have to examine them.

##### Insert Size Metrics
These files are located in the `InsertSizeMetics/` directory, and contain
per-sample insert size summaries. It is only generated if the data are
paired-end. We will not examine these files for the tutorial, but they will
be useful if you plan to submit your data to a database that requires summaries
of the insert sizes, such as NCBI GEO.

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
see the shapes of the distributions. Here, we see that each sample has a high
density of relatively lowly-expressed genes, a high density of
moderately-expressed genes, and a long tail of very highly-expressed genes.

An example of the heatmap of high variance genes is shown below.

![Heatmap]({{ "/graphics/rnaseq_cmd/high_variance_heatmap.png" | prepend: site.baseurl }})

The blue and red bars that correspond to sample groups will not be present if
you did not supply experimental groups. This plot shows samples as columns and
genes as rows. Samples are clustered based on similarity, with the distance
relationships shown with the dendrogram. Gene expression (CPM) is displayed as
rows, with red colors corresponding to low expression and yellow/white colors
corresponding to high expression. Only the 500 genes with the highest variance
among samples are shown.

An example of the MDS plot is shown below.

![MSD]({{ "/graphics/rnaseq_cmd/mds_plot.png" | prepend: site.baseurl }})

MDS is a way to identify factors that explain similarities and dissimilarities
among the samples. Samples that are more similar (have lower distance) appear
closer together on the plot. In this case, samples from the same treatment group
are more similar on average, than samples from different treatment groups. The
exception is sample `GrpA.0.15`, which has high distance from every other
sample. This makes sense, because in the CPM plot, its distribution of
expression values is qualitatively different from each other sample.

## Part 3: Detailed Description of Pipeline Steps
This section contains a detailed description of all constituent steps of the
`bulk_rnaseq` pipeline. Steps **1**, **2** and **3** occur in a task
array, with one task per sample. Steps **4** and **5** occur in in a single
job that processes the output from each single-sample task.

### 3.1: Summarizing Read Quality
The first part of the pipeline uses `fastqc` to generate summaries of read
length, base quality, base composition, and non-biological sequence
contamination. These reports are written to the working directory for each
sample. Both the HTML report and the text report (in the FastQC-produced ZIP
file) are made available. For more information on how to read the quality
reports generated by FastQC, please refer to our other tutorial that treats
Illumina data quality control in depth: https://pages.github.umn.edu/MSI-RIS/Tutorials/illumina_galaxy/.

This step is always run. You should view the FastQC reports to see if there are
technical explanations for patterns you observe in your results.

### 3.2: Cleaning Reads
The second part of the pipeline uses Trimmomatic to clean low quality bases,
adapter contamination, and reads that fail length filters from your input
datasets. This step is optional - there are arguments in favor of and against
cleaning reads. To disable read cleaning, simply add the `--no-trim` option to
the `gopher-pipelines.py bulk_rnaseq` command. We enable cleaning by default
because it is relatively fast and in most cases, will not negatively impact a
differential expression analysis.

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
5. After all adatper and quality trimming, remove reads that are shorter than
   18 bases.

It is possible to override these options by supplying the
`--trimmomatic-opts=" ... "` option to the `gopher-pipelines.py bulk_rnaseq`
call. Please note that any options you supply will **totally override** the
defaults that we set. If you want to only change one of our options, then you
must specify the entire options string with your desired change inserted. Please
also note that if you supply alternative options to Trimmomatic, then the onus
is on you to ensure that they are valid and produce the desired output. We
cannot guarantee that any combination of options outside of our defaults will
work properly for any given dataset. Please consult the Trimmomatic
manual for a full description of available options.

After Trimmomatic cleaning, FastQC is run on the cleaned reads. You can then
compare the raw and the processed reads to determine if you need to adjust any
of the parameters for cleaning.

### 3.3: Mapping Reads
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
   the `--ppn` option to `gopher-pipelines.py bulk_rnaseq`)

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

### 3.4: Counting Reads in Genes
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

### 3.5: Filtering and Differential Expression Testing
The raw counts matrix generated by `featureCounts` is then processed with a
R script that filters the counts matrix, generates summary plots, calcualtes a
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
with data analysis that falls outside of the `gopher-pipelines`.

## Part 4: Pipeline File Formats
The `bulk_rnaseq` pipeline of `gopher-pipelines` generates up to three
pipeline-specific file formats. Technically, these files are not proprietary
in any way, and can be used by third-party applications and custom user scripts.
However, we cannot support the diversity of ways that people may interact with
the internal files of `gopher-pipelines`.

### 4.1: Experimental Groups Template
This is a comma-separated values (CSV) file that contains placeholders for the
experimental for the samples in question. By default, it contains only two
fields, `SampleName` and `Group`. The `SampleName` field contains the inferred
name of the sample, derived from the filename of the FASTQ file. The `Group`
column is auto-populated with a `NULL` value. We do not make any attempt to
automatically assign samples to groups based on patterns in the filenames.

## Part 5: More Complex Analyses

## Part 6: Recommendations
- design experiments with power (controls and replicates)
- be careful with public data (cf rnalater)
- "best-in-class" software
  - HISAT2
  - Kallisto (not generally)
  - HTSeq
  - EdgeR, DESeq2
- Be careful with tools built for microarrays


## Part 7: Other RNASeq Applications
- Coexpression (WGCNA)
- Tx assembly (including splicing)
- Variant discovery (in absence of reference genome)
