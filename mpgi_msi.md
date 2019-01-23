---
layout: default
title: MPGI-MSI Workshop
permalink: /mpgi_msi/
exclude: false
updated: 2019-01-22
delivered: 2019-01-11
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

This is a warning. Common pitfalls, cautionary information, and important points
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
By the end of this workshop, you should

- Be familiar with the structure of MSI systems
- Know how to get an MSI account and know the default storage and compute allocations
- Know how to get assistance with using MSI systems
- Have a working definition of what constitutes a "genomics project"
- Understand the difference between a "broad" and a "deep" sample
- Know the difference between Illumina short read sequencing, Pacific Biosciences SMRT sequencing, and Oxford Nanopore sequencing
- Log into MSI systems and access a compute node
- Be able to write a basic PBS job script for use on MSI
- Understand the job resource request and edit it to suit your needs
- Be able to choose a system and queue to run your job
- Be able to submit jobs to the compute queues
- Understand the difference between processors/cores and nodes
- Know the two basic methods for parallelization on MSI systems

The modules in this workshop do not go into exhaustive detail on any of these
topics. For more detailed information, please come to the other MSI tutorials.
You can view a schedule of the tutorials we offer [here](https://www.msi.umn.edu/tutorials).

[Return to top](#top)
### <a name="0.3"></a> 0.3: Scope of the Workshop
This workshop is at the introductory level to acquaint new users with MSI
resources and basic genomics concept. The material linked to in the later
sections of this document is more detailed and advanced. The breakdown of
content presented in each of the modules is as follows:

#### Module 1: Introduction to MSI
- Services that MSI provides
- Physical location of MSI
- Structure of MSI systems
- Data storage service and policies
- High performance computing service and policies
- Job queues
- Consulting and support services
- Eligibility

#### Module 2: Introduction to Genomics Techniques and Terminology
- Examples of genomics questions
- Common technologies in genomics
- Depth vs. breadth
- Modern types of sequencing
- Variants of DNA and RNA sequencing
- Considerations for genomics project design
- Resources at MSI for genomics

#### Module 3: Intro to PBS and Job Submission
- What is PBS?
- MSI system structure
- Connecting to MSI systems
- Writing a job script
- PBS email reports
- Modifying a resource request

#### Module 4: Parallelization with GNU Parallel and Task Arrays
- Difference between serial and parallel processing
- Tools available on MSI for parallelization
- How to choose parallel tools
- Using GNU Parallel
- Using task arrays
- GNU Parallel with multiple nodes

## <a name="1"></a> Module 1: Introduction to MSI
### Slide Deck
You can view the slide deck [here](https://drive.google.com/open?id=1WWM2S9kYtWeT2rT9fMchb7VLqBUhI0Y3XE2S1U1WiX4).

### Supporting Material
#### Links to MSI Services
- [Storage, HPC, and consulting services](https://www.msi.umn.edu/services)
- [Getting MSI access](https://www.msi.umn.edu/content/eligibility-getting-access)
- [Storage allocation policies](https://www.msi.umn.edu/content/storage-allocations)
- [HPC service](https://www.msi.umn.edu/content/hpc), including service unit (SU) and priority ("FairShare")
- [Tutorial schedule](https://www.msi.umn.edu/tutorials)
- [List of available software](https://www.msi.umn.edu/software)

#### Links to MSI System Info
- [Names and limits of queues](https://www.msi.umn.edu/queues)
- [System status](https://status.msi.umn.edu/)
- [Queue usage](https://www.msi.umn.edu/qstat), dynamically displays queue loads


## <a name="2"></a> Module 2: Introduction to Genomics Techniques and Terminology
### Slide Deck
You can view the slide deck [here](https://drive.google.com/open?id=1FrTF8Kp6j3sCJq0bgkGvgqLqYQ-5cj9eLmxV3IiHdAs).

### Supporting Information
#### Genomics Technologies
High-throughput sequencing is one of the most common technologies used in
genomics studies. Many studies derive results from a variant of a nucleic acid
sequencing protocol, which is why we cover it extensively in the presentation.

Other genomics technologies that are also in common use involve hybridization
to an array and measurement of a fluorescent signal. These may be used in
single nucleotide variant genotyping (e.g., Illumina Infinium arrays), gene
expression assays (e.g., Affymetrix GeneChip arrays), and DNA methylation
levels (e.g., Illumina Methylation arrays).

More specialized genomics technologies include instruments such as the
nanoString nCounter Analysis System, which uses a variation of microarray
technology to directly count the number of nucleic acid molecules in a sample.
Bionano instruments hybridize probes to long DNA molecules and visualize them
in a similar way to FISH to generate optical maps, which enable detection of
large structural variants. There are many more genomics technologies and
instruments available for your research. As always, the best way to know which
technologies suit your research goals is to read current literature.

#### Depth vs. Breadth
It is important to consider a genomics experiment in terms of *depth* and
*breadth*. The *depth* of sampling refers to the intensity of data collection
from each sample. The *breadth* of sampling refers to the number of biological
or technical samples that are included in the study. With a constrained budget,
there is a trade-off between depth and breadth.

![Depth and breadth]({{ "/graphics/mpgi-msi/Depth-Breadth.png" | prepend: site.baseurl }})

Projects such as genome assembly require very deep sampling, but not very broad
sampling. Ideally, multiple technologies or multiple independent libraries from
the same individual are used for genome assembly.

Transcriptome assembly does not require as deep sampling because the target for
assembly is much smaller than for genome assembly, but it is helpful to have
slightly higher breadth. Samples from multiple time points, multiple tissues,
or multiple environmental conditions aid in transcriptome assembly by capturing
rare and conditionally expressed transcripts.

Studies that examine variation among multiple samples, such as differential
gene expression studies or genetic diversity studies, should focus more on
breadth than depth of sampling.

Detailed material for handling specific types of genomic data is covered by
our other [tutorials](https://www.msi.umn.edu/tutorials). For now, we offer
a tutorial about RNAseq analysis, but we plan to include other genomics
analysis soon.

#### University of Minnesota Genomics Center (UMGC)
The University of Minnesota has a core facility that provides genomics services.
The link to the UMGC site is [here](http://genomics.umn.edu/).

#### Long vs. Short Read Sequencing
Illumina (and 454 and Ion Torrent) sequencing are referred to as "short read"
sequencing. Read lengths from Illumina instruments are between 36 and 600
base pairs, with every read having a fixed read length dependent on the
instrument and desired run type. Per base of sequence data, short read
sequencing is very cheap and generates millions of reads. Each read has
relatively low error rates, approximately 1 in 400 ([Pfeiffer et al. 2018](https://www.nature.com/articles/s41598-018-29325-6)).

Reads from Pacific Biosciences SMRT sequencing and Oxford Nanopore sequencing
are referred to as "long read" sequencing reads. They can be on the order of
tens of kilobases long (e.g., [Hengyun et al. 2016](https://www.sciencedirect.com/science/article/pii/S1672022916301309)),
which helps with assembly of large and repetitive genomes. Long reads can span
repeated elements, which resolves the orientation and copy number of the
sequence. Long reads are error-prone, with approximately 1 in 20 error rates
(e.g., [Au et al. 2012](https://journals.plos.org/plosone/article?id=10.1371/journal.pone.0046679)).
However, the errors are putatively random ([Robers et al. 2013](https://genomebiology.biomedcentral.com/articles/10.1186/gb-2013-14-6-405)),
which means that base confidence continually increases with increasing coverage,
unlike short reads, which have biased error profiles. A common approach is to
"patch" errors in long reads with short read sequencing data
(e.g., [Zhang et al. 2018](https://onlinelibrary.wiley.com/doi/full/10.1111/tpj.14120)).

<div class="warn" markdown="1">

#### Considerations for Genomics Projects
- GIGO: "Garbage in, garbage out"

    Low-quality inputs (garbage in) will produce low-quality results (garbage
    out). This is a principle that was lifted over from computer science. The
    meaning is not exactly the same in genomics as in computer science, but the
    concept is still relevant. The sequencing facility cannot generate
    high-quality sequence data from low-quality samples, and a genomics analysis
    workflow cannot generate high-quality results from low-quality data.

- Trade-offs between cost and effort

    Techniques that save on money up-front, such as reduced representation
    methods, often require more difficult analytical techniques to properly
    handle. Be sure to budget extra time and effort for your projects if you
    use a specialized protocol.

- KEEP NOTES!

    Keep notes from the very beginning of your project. Sample metadata can be
    used as covariates when running your analyses to model variation from
    technical sources. An example list of metadata to keep would be

    - Collection site and conditions
    - Collection time
    - Extraction protocol
    - Concentration
    - RIN or DIN scores
    - Who performed the extractions
    - Sample storage conditions
    - Bench/flat/cage/plate of sample rearing
    - Library preparation protocol
    - Flowcell ID
    - Lane number

    [Karl Broman and Kara Woo](https://www.tandfonline.com/doi/full/10.1080/00031305.2017.1375989)
    have described a way to keep data that ensures that your values are properly
    formatted and that analysis goes as smoothly as possible.

    Additionally, maintain your computational analyses as scripts with detailed
    comments. Your collaborators, your supervisor, your peer reviewers, and
    most importantly, your future self will need to know how you performed an
    analysis. Be sure your scripts document at least the following

    - Paths to data files
    - Software names and versions
    - Output file names
    - Time and date that analyses were run
    - Author of script

    A good scripting principle is that the script should be readable by
    humans, but run on computers. You colleagues should be able to easily read
    your script to know what it is doing, and it should run without throwing
    errors.


</div>

## <a name="3"></a> Module 3: Introduction to PBS and Job Submission
### Slide Deck
You can view the slide deck [here](https://drive.google.com/open?id=14-QitVjoM7uOo2QPNqWPD7TD2vPP3YzR5FicdtimCho).

### Supporting Material
#### How to Connect to MSI Systems
MSI maintains [a guide](https://www.msi.umn.edu/content/connecting-hpc-resources)
for connecting to MSI systems. The instructions cover the required tools and
procedures for Windows, Mac OSX, and Linux workstations.

<div class="warn" markdown="1">

If you are connecting from off-campus, you will need to connect through the
university VPN. UMN OIT maintains a [page](https://it.umn.edu/downloads-guides)
with download links, installation guides, and setup instructions for using the
University of Minnesota VPN.

</div>

#### A Reminder: Structure of MSI Systems

![MSI Systems]({{ "/graphics/mpgi-msi/Systems.png" | prepend: site.baseurl }})

When you first connect to MSI systems, you will be connected to a `login`
system. This is a system that serves as a common gateway to the HPC systems,
which are more isolated from the Internet.

From the `login` system, you must use `ssh` to connect to one of the HPC systems
listed in the diagram above. When you connect to the HPC system, you will be
connected to a login node for the HPC system. These are different from the
`login` system that you connect to when you first access MSI servers.

<div class="warn" markdown="1">

Do not run computationally intense tasks on the HPC login nodes. You run the
risk of crashing the node and impacting availability for other people on
campus. This behavior has resulted in account restrictions in the past.

</div>

#### Job Queues
Follow [this link](https://www.msi.umn.edu/queues) to view information about
the job queues on the HPC systems. Use the job queues to run analyses in a
**non-interactive** way. You can submit the job, and it will run on the cluster
after you log out.

#### Writing a Job Script
MSI maintains [a guide](https://www.msi.umn.edu/content/job-submission-and-scheduling-pbs-scripts)
for writing job scripts and submitting them to the various queues. We will
digest the job script format here. The format is a standard Bash script with
a few special lines at the top of the file that declare the parameters for the
job. These special lines (called "directives") start with `#PBS` and follow
a special syntax for requesting resources. The header from the slide deck is
pasted below:

```
#!/bin/bash
#PBS -l nodes=1:ppn=1,mem=2gb,walltime=1:00:00
#PBS -m abe
#PBS -M YOUR_X.500@umn.edu
#PBS -q lab
```

<div class="warn" markdown="1">

All lines in the job script are case-sensitive, which means that capitalization
is important. a is different from A.

Spaces are also important. Pay attention to where the spaces are in the example
script!

</div>

Each line will be desribed below:

1. `#!/bin/bash`

    This line is a special line that tells the operating system that the file
    is a bash script. Specifically, the `#!` part is a special character that
    tells the system to use the program that comes after it to run the script.
    In this case, we specify `/bin/bash`, which is the standard Bash shell
    interpreter.

2. `#PBS -l nodes=1:ppn=1,mem=2gb,walltime=1:00:00`

    This line contains a PBS directive, denoted by the `#PBS` at the beginning
    of the line. The `-l` (lowercase L) flag means that this line contains a
    resource request. The resource request does not have any spaces! In this
    case, we are requesting one node (`nodes=1`), with one processor per node
    (`ppn=1`). Note the colon (`:`) separating the `nodes=1` and `ppn=1`. The
    next part of the request is a total of `2gb` of memory (`mem=2gb`). We are
    also requesting a total of 1 hour of runtime (`walltime=1:00:00`). The
    format for the walltime request is `HH:MM:SS`, where `HH` is hours, `MM` is
    minutes, and `SS` is seconds.

3. `#PBS -m abe`

    This is another directive that specifies the email notification settings.
    In this case, we are requesting an email notification be sent when the
    job **a**borts with an error (`a`), when the job **b**egins (`b`), and
    when the job **e**xits successfully (`e`).

4. `#PBS -M YOUR_X.500@umn.edu`

    This is another directive that specifies the email address to which to send
    notifications. Note that this is a placeholder value in the example. Please
    specify your own email address in the job script.

5. `#PBS -q lab`

    Finally, this directive specifies the queue to which you would like to send
    your job. In this example, we are using the `lab` queue. Please refer to
    our [queues page](https://www.msi.umn.edu/queues) to see the full list of
    available queues. Remember that not all queues are available on all systems;
    you must be submitting from a Measabi login node to run jobs in the
    `mesabi` queue, for example.

The directives can also be passed as options to `qsub` when you submit a job.
The lines starting with `#PBS` could be omitted from you job script, and the
job could be sent in exactly the same way by giving this command:

```
qsub -l nodes=1:ppn=1,mem=2gb,walltime=1:00:00 -m abe -M YOUR_X.500@umn.edu -q lab
```

I prefer to keep the directives written into the script so that I have a log of
the size of the request I made. See the warnings in the Module 2 section above.
If both `qsub` command line options and `#PBS` directives are given in the
script, the `qsub` command line options take precedence.

<div class="info" markdown="1">

Job scripts are Bash scripts. If you would like to learn more about Bash
scripting, there are many excellent guides available. One great introductory
tutorial to scripting and the GNU/Linux command line is maintained by the
[UC Riverside High Performance Computing Center](https://hpcc.ucr.edu/manuals_linux-basics_intro.html).

For more advanced Bash scripting, I like the
[Advanced Bash Scripting Guide](https://www.tldp.org/LDP/abs/html/), which
contains many examples of sophisticated scripting techniques.

</div>

#### Reading a PBS Job Email
When your job exits, you will get a cryptic-looking email that describes the
results of your job. An example job email is pasted below:

```
PBS Job Id: 762559.nokomis0015.msi.umn.edu
Job Name:   push_LA.job
Exec host:  labc01.msi.umn.edu/0
Execution terminated
Exit_status=0
resources_used.cput=01:48:40
resources_used.vmem=422480kb
resources_used.walltime=06:19:51
resources_used.mem=14396kb
resources_used.energy_used=0
req_information.task_count.0=1
req_information.lprocs.0=1
req_information.total_memory.0=4194304kb
req_information.memory.0=4194304kb
req_information.thread_usage_policy.0=allowthreads
req_information.hostlist.0=labc01.msi.umn.edu:ppn=1
req_information.task_usage.0.task.0={"task":{"cpu_list":"0","mem_list":"0",
"cores":0,"threads":1,"host":"labc01.msi.umn.edu"}}
Error_Path: labc03.msi.umn.edu:/panfs/roc/groups/14/msistaff/konox006/push_LA.job.e762559
Output_Path: labc03.msi.umn.edu:/panfs/roc/groups/14/msistaff/konox006/push_LA.job.o762559
```

This is more information than you really need, so we will highlight the key
pieces here.

- `PBS Job Id: 762559.nokomis0015.msi.umn.edu`

    This is the job ID in our system logs. If you request help or
    troubleshooting for a particular job, you may be requested to send this
    information.

- `Job Name:   push_LA.job`

    This is the job name. You can specify any name you want here, but I usually
    leave it as the name of the script, which is the default.

- `Exec host:  labc01.msi.umn.edu/0`

    This is the name of the machine that ran your job. If you experience strange
    issues, such as file permissions being incorrect or corrupted output files,
    you may be asked to send this information for identifying troublesome
    hardware.

- `Execution terminated`

    This just means the job finished.

- `Exit_status=0`

    This means the job finished successfully. Technically speaking, this is the
    *exit status* of the script when it was run. By UNIX convention, an exit
    status of `0` is a success. `1` is a general error. Other numbers are
    generally program-specific, so you will have to dig into your script's log
    files to see what went wrong.

- `resources_used.cput=01:48:40`

    This is the amount of CPU-time (CPU ticks * time) that was used. By itself,
    it is not very useful information.

- `resources_used.vmem=422480kb`

    This is the amount of *virtual address space* used by the job. This is the
    maximum amount of virtual memory (including RAM and temporary swap space
    on disk) that was used by the job.

- `resources_used.walltime=06:19:51`

    This is the amount of walltime (real time) that was used by the job. This
    directly corresponds to the `walltime=` part of the request.

- `resources_used.mem=14396kb`

    This is the maximum amount of *physical memory* used by job. When you are
    tuning the resource request string, this value is the important one in
    deciding how much memory to request.

- `Error_Path: labc03.msi.umn.edu:/panfs/roc/groups/14/msistaff/konox006/push_LA.job.e762559`

    This is the path to the *error file*. This does not necessarily mean that
    your job encountered errors. Rather, it contains the text that is written
    to the *standard error* output channel, which is used for warnings and
    error messages, by convention. You can check this file for progress messages
    or debugging messages for your job.

- `Output_Path: labc03.msi.umn.edu:/panfs/roc/groups/14/msistaff/konox006/push_LA.job.o762559`

    Similarly, this path contains the text that was written to the
    *standard output* output channel. This does not necessarily mean that your
    job finished successfully. It is also not necessarily the results of your
    analysis.

#### q-Commands Documentation
MSI servers run PBS TORQUE from Adaptive Computing. You can view the manual
for the various commands mentioned (`qsub`, `qstat`, etc.)
[here](http://docs.adaptivecomputing.com/torque/4-0-2/Content/topics/12-appendices/commandsOverview.htm).
Please note that not all commands are available on MSI systems.

## <a name="4"></a> Module 4: Parallelization with GNU Parallel and Task Arrays
### Slide Deck
You can view the slide deck [here](https://drive.google.com/open?id=1eD-qoXzRS2YI_MqI6OzVpNwA_IXHx07WXd_VM-6mQM8).

## Feedback
This tutorial document was prepared by Thomas Kono, in the RIS group at MSI.
Please send feedback and comments to konox006 [at] umn.edu. You may also send
tutorial delivery feedback to that address.

[Return to top](#top)
