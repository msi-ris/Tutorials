---
layout: default
title: Illumina Data QC Tutorial With Galaxy
permalink: /illumina_galaxy/
exclude: false
updated: 2018-06-13
delivered: 2018-06-13
---

# Illumina Data Quality Control With Galaxy
*Last Updated: {{page.updated}}*

*Last Delivered: {{page.delivered}}*

This tutorial is also available as a [PDF]({{"/materials/illumina_qc_galaxy/Illumina_QC_Galaxy_Current.pdf" | prepend: site.baseurl }}).

## Part 0: Introduction
### What is Galaxy?
Galaxy is a web-based interface that allows users to create complex
computational pipelines to analyze biological data. Galaxy is designed to help
you create reproducible workflows that can be used with multiple datasets,
shared with others and published. Common bioinformatics software such as BLAST,
BWA and GATK can be accessed though the Galaxy interface along with many other
tools for converting between different formats, manipulating data and basic
statistics.

### Galaxy at MSI
There are many instances of Galaxy, the one available to you though MSI is
maintained by MSI and connects directly to the computational resources at MSI.
The tools available will vary depending on which instance of Galaxy you use.
While transferring workflows from one instance of Galaxy to another is easy, MSI
has no control over which tools are available in other Galaxy instances. If
there is a tool that you have used in a different instance of Galaxy that is not
available in the MSI instance send a request to help@msi.umn.edu.

### Scope of This Tutorial
- Give participants experience with the basic functionality of Galaxy
    - Accessing Galaxy at MSI
    - Galaxy layout
    - Loading files into current Galaxy history
    - Creating a workflow
    - Sharing histories and workflows
    - Where to get more information
- Basic processing and quality control on Illumina sequencing data
    - Evaluating read quality
    - Adapter removal
    - Low quality read removal
    - Read trimming

### Where to Get More Information
- Other Galaxy users: https://wiki.galaxyproject.org/
- MSI: https://www.msi.umn.edu/content/bioinformatics-analysis

## Part 1: Getting Started With Galaxy
### Sections of Galaxy
Galaxy has three main sections:
- Tools pane
    - Found on the left side of the browser
    - Contains all of the tools that can be used with Galaxy, including those
      that do simple text manipulations and arithmetic to those that do complex
      functions specific to the analysis of NGS data and statistics.
    - Combinations of tools allow you to analyze your data.
    - Tools are organized by category, or can be found by the search bar.
- Histories pane
    - Found on the right side of the browser
    - Contains the history of the tools you have used, and the results from
      those tools
    - Histories can be saved, shared, and turned into workflows that can *also*
      be savd, shared, and reused.
- Center pane
    - Found in the center of the browser
    - When using a tool, the options for that tool and information about the
      tool will be in the center pane.
    - Clicking on the *eye icon* in the history pane will open a view of the
      data in the center pane.

### Accessing Galaxy at MSI
Open a web browser and navigate to the MSI Galaxy galaxy.msi.umn.edu. Log in
with your MSI username and password

The labeled items are as follows:

1. Tools Pane
2. Center pane
3. History Pane
4. The side panels can be collapsed via arrows in the bottom corner to provide a
   better view of the center pane.
5. Search bar to find tools.
6. The total quantity of data your group has stored in Galaxy is displayed in
   the top right corner.
7. You can always get back to the main screen using *Analyze Data* in the top menu
   bar

### Import FASTQ Files from Data Library
#### Getting Data Into MSI Galaxy
##### Sequencing Data from the UMGC
Sequencing data from UMGC can be accessed in Galaxy though the creation of a
data library. In general, each PI with access to Galaxy account will have one
data library that can contain many different pieces of data. When you or your PI
receives an email from UMGC indicating that your sequencing data is available
you can have that data moved into your PI's Galaxy data library by forwarding
the email to help@msi.umn.edu with a request to add the data to Galaxy. You can
then access the sequencing data library from the *Shared Data* tab in the blue
bar at the top of the Galaxy page. If your PI doesn't currently have a data
library a new one will be created the first time you request to have data added
to Galaxy.

##### External Data
The *Get Data* heading in the Tool Pane is a good resource for obtaining
external data from public databases such as the UCSC genome browser and SRA.
You can also upload small (<2GB) files directly from your computer. When data is
uploaded using the tools under Get Data they will appear in your current
history.

##### Large External Datasets
Data files that are larger than 2GB will have to be placed into a data library
to be accessed in Galaxy. In your groups home directory there is a galaxy folder
(`/home/yourGroup/galaxy`). To get data into your PI's data library move it into
the galaxy folder in your groups home directory then send a ticket to
help@msi.umn.edu with the location of the data to be added to your PI's data
library.

## Part 2: Evaluating FASTQ Quality
### FASTQ Format and Quality Scores
This tutorial is geared towards Illumina data in FASTQ format, other sequencing
methods (i.e., Roche 454) may produce reads with a different patterns of errors
or a different file format. Quality control tools for other NGS data types can
be found under the NGS: QC and manipulation heading in the Tool Pane.

A sequence record in a FASTQ file consist of four lines:
1. an `@accession` line
2. sequence data
3. `+` place holder line
4. quality score line

FASTQ quality scores encode the estimated chance of a miscalled base at each
location. Single ASCII characters are used to encode the quality scores, as
opposed to raw numbers, so that there is always a 1-to-1 relationship between
the number of bases in the read and the length of the quality score. Quality
score reflect the probability that a base call was incorrect, calculated as a
Phred quality score ( Phred Q = -10log(p), where p isthe probability that the
inferred base is incorrect). The higher the Phred score the smaller the
probability that the base call was incorrect. A Phred score of 10 indicates a 1
in 10 chance of an incorrect base call while a oared score of 50 indicates a 1
in 100,000 chance of an incorrect base call.

Unfortunately, FASTQ files from different sources sometimes encode quality
scores slightly differently. Sanger and current Illumina FASTQ format uses a
Phred+33 encoding, which means that the lowest Phred score of 0 is encoded as
ASCII character 33 (!), while Solexa and pre-2012 Illumina software uses
Phred+64 encoding (Phred 0 encoded as @). But in all cases, the higher the Phred
quality scores the higher quality the base call. In Galaxy you can use FASTQ
Groomer to ensure your data is in the Sanger/Illumina 1.8 + encoding

### FastQC Metrics
- Basic Statistics
  Gives the name of the input file, encoding used for the quality score, total
  sequence count, average sequence length and GC content (%).
- Per Base Sequence Quality
A important figure showing the average quality score at each position across all reads. In
general, quality scores are lower at the start and each of reads. Sudden dips in the middle
of a read can signify failed cycles in the sequencing run (machine errors).
- Per Tile Sequence Quality
Gives quality information based on the location of the read on the flowcell tile. A good plot
will be blue all over, red color indicates cells with low quality reads.
- Per Sequence Quality Scores
Histogram charting the average quality across a read. Low quality reads can be removed but
a majority (at least 75%) of you data should be of high quality.
- Per Base Sequence Content
The frequency of each nucleotide at each position across all of the reads. Extremely high
nucleotide bias can be a sign of trouble, short stretches with high bias can be caused by the
presence of linkers, barcodes or adapter contamination. There is usually some minor bias in
the first 11-13bp of RNA-seq data due to not-quite random hexamer sequence priming but
this bias is accounted for in the downstream analysis.
- Per Sequence GC Content
This figure will show you both the theoretical distribution of GC content and the GC content
of your data. These distributions should be similar.
- Per Base N Content
Rate of ambitious base calls (N) for each position along the reads. This count should be
very low (<10), to many N calls indicates issues with the sequencing run (usually machine
errors).
- Sequence Length Distributions
Histogram of the sequence lengths. Illumina reads that have not been trimmed will all have
the same length, once trimmed you want a majority of your reads to be full length and a
small percentage to be shorter.
- Sequence Duplication Levels
Frequency of exact sequence duplicates in the dataset. High duplication rates can be caused
by PCR artifacts and/or low library diversity. Low levels of duplication can be removed but,
high levels indicate issues with the library preparation.
- Overrepresented Sequences
A list of overrepresented sequences if they exist in the data. These are the sequences that
are contributing to the data in the Sequence Duplication Level graph.
- Adapter Content
While the Overrepresented Sequences analysis will pick up adapter contamination the results
can often be unclear. This plot looks specifically for a set of Illumina adapters and gives a
plot indicating the the total proportion of your lib ray which contain these adapters.
- K-mer Content
Shows the amount (% of reads) and sequence of overrepresented K-mers. High levels of
overrepresented sequences usually arise from adapter contamination and these levels should
drop after adapters are removed from your data.

## Part 3: Cleaning FASTQ Datasets
## Part 4: Evaluate Cleaned FASTQ Quality
## Part 5: Evaluating and Cleaning Paired-End Data
## Part 6: Workflows
## Part 7: Sharing Workflows and Histories
## Part 8: Cleaning up Histories and Deleting Data from Galaxy
