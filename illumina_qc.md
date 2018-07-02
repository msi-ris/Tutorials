---
layout: default
title: Illumina Data QC Tutorial With Galaxy
permalink: /illumina_galaxy/
exclude: false
updated: 2014-09-16
delivered: 2018-06-13
---

# Illumina Data Quality Control With Galaxy
*Last Updated: {{page.updated}}*

*Last Delivered: {{page.delivered}}*

This tutorial is also available as a [PDF]({{"/materials/illumina_qc_galaxy/Illumina_QC_Galaxy_Current.pdf" | prepend: site.baseurl }}).

## Part 0: Introduction
### Formatting in This Document
Throughout this tutorial, there will be formatting cues to highlight various
pieces of information.

<div class="info" markdown="1">

This is just information. There are no tutorial-related tasks in these boxes.
We will write background or supporting information like this.

</div>

```
This is code, or a literal value that you must enter or select to run a part
of the tutorial
```

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
available in the MSI instance send a request to <help@msi.umn.edu>.

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
- Other Galaxy users: <https://wiki.galaxyproject.org/>
- MSI: <https://www.msi.umn.edu/content/bioinformatics-analysis>

## Part 1: Getting Started With Galaxy

<div class="info" markdown="1">

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

</div>

### Accessing Galaxy at MSI
Open a web browser and navigate to MSI Galaxy (<https://galaxy.msi.umn.edu>).
Log in with your MSI username and password.

The labeled items are as follows:

1. Tools Pane
2. Center pane
3. History Pane
4. The side panels can be collapsed via arrows in the bottom corner to provide a
   better view of the center pane.
5. Search bar to find tools.
6. The total quantity of data your group has stored in Galaxy is displayed in
   the top right corner.
7. You can always get back to the main screen using `Analyze Data` in the top menu
   bar

### Import FASTQ Files from Data Library
#### Getting Data Into MSI Galaxy

<div class="info" markdown="1">

##### Sequencing Data from the UMGC
Sequencing data from UMGC can be accessed in Galaxy though the creation of a
data library. In general, each PI with access to Galaxy account will have one
data library that can contain many different pieces of data. When you or your PI
receives an email from UMGC indicating that your sequencing data is available
you can have that data moved into your PI's Galaxy data library by forwarding
the email to <help@msi.umn.edu> with a request to add the data to Galaxy. You
can then access the sequencing data library from the `Shared Data` tab in the
blue bar at the top of the Galaxy page. If your PI doesn't currently have a data
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
<help@msi.umn.edu> with the location of the data to be added to your PI's data
library.

</div>

1. At the top of the screen select `Shared Data` then in the menu 
   `Data Libraries`.
2. Select `QC Illumina Data` from the list of data libraries.
3. Expand the `FastQ` folder and check the boxes next to the first file,
   `RNA-Seq.fastq`.
4. Select `Go` next to `Import` to current history below the data files to move
   the data to your current history.
5. Select `Analyze Data` in the blue bar to move back to the main Galaxy view.

#### Set File Attributes

<div class="info" markdown="1">

##### Attributes
Setting the file attributes will tell the different tools in Galaxy what format
the data is in. Galaxy does some work to auto detect the files that can be used
as inputs for different tools. If you find that the file you want to use as an
input is not available in a drop down menu check to see if you have set the file
attributes. Information about different files types can be found though the USCS
genome browser and from Current Protocols in Bioinformatics.

#####Special note about FASTQ
FASTQ files contain quality information for each sequenced base encoded using
the characters found in the fourth line of each block. The preferred encoding
for MSI Galaxy is Sanger. If you are looking at Illumina data created in 2012 or
later your FASTQ files already using the Sanger encoding. If you sequencing was
done before 2012 then you should use FASTQ Groomer to convert to the Sanger
encoding (Sanger & Illumina 1.8+).

</div>

1. In the *History Pane* click on the `pencil icon` next to `RNA-Seq.fastq`.
   This will bring up the files Attributes in the *Center Pane*.
2. This is a human dataset so select `Human hg19 in GATK canonical` in the drop
   down menu under `Database/Build:`. You can scroll or if you begin to type
   `hg19` then you will only see the options with `hg19` in the name.
3. Click `Save`
4. Switch to the `Datatype` tab by selecting it from the top of the
   *Center Pane*.
5. Select `fastqsanger` from the drop down menu. You can scroll or if you begin
   to type `fastq` then you will see the options with `fastq` in the name.
   **NOTE: do not select `fastqcsanger`.**
6. Click `Save`

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
ASCII character 33 (`!`), while Solexa and pre-2012 Illumina software uses
Phred+64 encoding (Phred 0 encoded as `@`). But in all cases, the higher the
Phred quality scores the higher quality the base call. In Galaxy you can use
FASTQ Groomer to ensure your data is in the Sanger/Illumina 1.8 + encoding

### FastQC Metrics
- **Basic Statistics**

  Gives the name of the input file, encoding used for the quality score, total
  sequence count, average sequence length and GC content (%).

- **Per Base Sequence Quality**

  A important figure showing the average quality score at each position across
  all reads. In general, quality scores are lower at the start and each of
  reads. Sudden dips in the middle of a read can signify failed cycles in the
  sequencing run (machine errors).

- **Per Tile Sequence Quality**

  Gives quality information based on the location of the read on the flowcell
  tile. A good plot will be blue all over, red color indicates cells with low
  quality reads.

- **Per Sequence Quality Scores**

  Histogram charting the average quality across a read. Low quality reads can be
  removed but a majority (at least 75%) of you data should be of high quality.

- **Per Base Sequence Content**

  The frequency of each nucleotide at each position across all of the reads.
  Extremely high nucleotide bias can be a sign of trouble, short stretches with
  high bias can be caused by the presence of linkers, barcodes or adapter
  contamination. There is usually some minor bias in the first 11-13bp of
  RNA-seq data due to not-quite random hexamer sequence priming but this bias is
  accounted for in the downstream analysis.

- **Per Sequence GC Content**

  This figure will show you both the theoretical distribution of GC content and
  the GC content of your data. These distributions should be similar.

- **Per Base N Content**

  Rate of ambitious base calls (N) for each position along the reads. This count
  should be very low (<10), to many N calls indicates issues with the sequencing
  run (usually machine errors).

- **Sequence Length Distributions**

  Histogram of the sequence lengths. Illumina reads that have not been trimmed
  will all have the same length, once trimmed you want a majority of your reads
  to be full length and a small percentage to be shorter.

- **Sequence Duplication Levels**

  Frequency of exact sequence duplicates in the dataset. High duplication rates
  can be caused by PCR artifacts and/or low library diversity. Low levels of
  duplication can be removed but, high levels indicate issues with the library
  preparation.

- **Overrepresented Sequences**

  A list of overrepresented sequences if they exist in the data. These are the
  sequences that are contributing to the data in the Sequence Duplication Level
  graph.

- **Adapter Content**

  While the Overrepresented Sequences analysis will pick up adapter
  contamination the results can often be unclear. This plot looks specifically
  for a set of Illumina adapters and gives a plot indicating the the total
  proportion of your library which contain these adapters.

- **K-mer Content**

  Shows the amount (% of reads) and sequence of overrepresented K-mers. High
  levels of overrepresented sequences usually arise from adapter contamination
  and these levels should drop after adapters are removed from your data.

### Running FastQC
1. From the Tools Pane select the MSI header.
2. Select the FastQC: Read QC tool.
3. Alternatively, use the search bar at the top of the tool pane to find FastQC.
4. Make sure that you are using FastQC:ReadQC (version 0.60).
5. Select the file to analyze from the drop-down menu: `RNA-Seq.fastq`
6. Rename the output file to something meaningful such as "Precleaning".
7. Select `Execute`

### Viewing and Understanding FastQC Results
1. In the History Pane select the Eye Icon next to the name of the output from
   using the FastQC: ReadQC tool. This will allow you to view the results in the
   Center Pane.
2. Scroll to the "Per base sequence quality".
3. Note how the quality of the reads drops towards the 3′ ends of the reads
4. The per tile sequence quality is all blue because the overall quality of the
   reads is good.
5. Scroll to "Per sequence quality scores".
6. The over all quality of the reads is high with a vast majority of the reads
   having quality scores over 30.
7. Scroll to "Sequence duplication levels".
8. Note the presence of a small number of highly duplicated reads. The
   duplicated sequences and counts can be found in the Overrepresented
   Sequences table.
9. The high count and percentage values for the top three sequences in the
   Overrepresented Sequences list represent adapter contamination in this
   dataset and need to be removed.
10. Scroll down to "Kmer content"
11. These very spiky regions of over represented Kmers are common and often are
    biological and not a data quality issue

## Part 3: Cleaning FASTQ Datasets
### Why is Cleaning Required?
#### Low Quality Tails and Failed Cycles
For a variety of reasons, including decay of reagents as they sit on the
sequencing machine, the quality of base calls tends to decrease as sequencing
progresses. As a result the 5′ ends will tend to have higher quality than the 3′
ends and forward reads will tend to have better quality than reverse reads. Low
quality base calls can impair the accuracy of mapping algorithms so it is
important to to remove them. Low quality tails can be removed though the removal
of the 3′ ends from all of the reads but, that would result in the removal of
many reads that were of higher quality as well. More sophisticated methods only
remove the tails that show evidence of low quality. In Galaxy there are tools
that can accomplish either style of read trimming. Cycles fail because of
sequencing machine error, such as failure to incorporate a base, or failure to
image a specific region. For most analysis failed cycles can be ignored as they
will not have large effects.

#### Adapter Contamination
Illumina libraries consist of the DNA of interest (green) with ligated adapter
(red + yellow) on the 5′ and 3′ ends to proved priming site for the sequencing
reactions. The forward adapter (left) provides a region that binds to the
Illumina flow cell plate (blue) and a region to which the sequencing primer
binds to start the sequencing reactions. The reverse adapter (right) has the
same structure with the addition of a barcode sequence (yellow). Adapter
contamination occurs when the DNA fragment of interest is shorter than the
length of the sequencing read. This results in the opposite primer included in
the sequence of the final read, leading to adapter contamination.

Removal of adapter contamination in Galaxy can be accomplished using CutAdapt
and the sequence of the primers.

### Remove Low Quality Tails and Adapter Contamination With Trimmomatic
1. Under the NGS: QC and manipulation header select Trimmomatic
2. Select `Single End Mode` in the top drop down menu.
3. Select `RNA-Seq.fastq` from the second drop down menu.
4. Trimmomatic allows you to build a list of modules to complete all of your
   clean-up steps at once.
5. The first module `ILLUMINACLIP` will identify and remove adapter contamination.
6. Select `TruSeq2-SE` from the adapters drop down menu.
7. To add another module select Add new Task at the bottom of the module.
8. Select `LEADING` from the task drop down menu, the page with refresh and new
   options associated with the `LEADING` module will appear. Use the default
   settings.
9. Next add the `TRAILING` module, `SLIDINGWINDOW` and the `MINLEN` module. The
   default settings will work for all of these modules.
10. Once all of the modules have been added select `Execute`.

## Part 4: Evaluate Cleaned FASTQ Quality
In this section we will compare the results from FastQC between the original
FASTQ file and the quality and adapter trimmed FASTQ files. You should always
examine the results post FASTQ file clean up before moving forward with more
complex analysis. Here we will be showing both the results from the original
FASTQ files as well as those from the quality and adapter trimmed FASTQ files.

1. Use FastQC to examine the quality statistics for the quality and adapter
   trimmed FASTQ files. Select the Eye Icon to view the results.
2. Scroll down to "Per base sequence quality". Note the improvement in the
   average quality of the read tails.
3. Scroll down to "Sequence Length Distribution". Note the degree of trimming
   that has occurred. A vast majority of your data is still full length with
   only a very limited number of sequences that are between 36 and 40bp.
   Changing the `MINLEN` settings during trimming will change this distribution.
4. Scroll down to "Sequence Duplication Levels". Note the removal of the highly
   repeated sequences.
5. Scroll down to the list of Overrepresented Sequences. The percentage of your
   data that is an overrepresented is now greatly reduced.

## Part 5: Evaluating and Cleaning Paired-End Data
This section will guide you though using FastQC and Trimmomatic on paired-end
data.

1. Using the gear icon at the top of the History Pane create a new history.
2. Select Shared Data then Data Libraries from the top header bar.
3. Select QC of Illumina Data from the list of data libraries. Remember the
   search bar!
4. Import `Tutorial file R1.fastq` and `Tutorial file R2.fastq` into your new
   history.
5. Run FastQC on the new fastq files. You will have to run the tool on each
   fastq individually.
6. Use Trimmomatic in Paired End Mode with TruSeq2-PE adapters to trim the data.
   Use the `ILLUMINACLIP`, `LEADING`, `TRAILING`, `SLIDINGWINDOW` and `MINLEN`
   modules with the default settings.
7. Notice there are 4 Trimmomatic outputs: forward paired, forward unpaired,
   reverse paired and reverse unpaired.
8. Run FastQC on the forward paired and reverse paired data to see the effect
   of your trimming.

## Part 6: Workflows
### Creating a Workflow
The ability to create, reuse, share and publish workflows is on of Galaxy's
largest strengths. Creating workflows allows you and anyone you want to
collaborate with to exactly recreate analysis. You can think of workflows as
your computational lab notebook, they are how you document your computational
work. Workflows are also handy when you have to clean up your Galaxy space.
Saving the raw input data and the workflow that leads to a final result allows
you to delete the intermediate files yet retain the ability to recreate the
entire analysis at any time. Workflows can be extracted from histories or
created from scratch. Either method will result in a useable workflow so how you
choose to build one is up to you.

Workflows are made up of connected tools, each tool is represented as a box and
data moving from one tool to another is represented by the arrows. The inputs
required for the tool can be found above the horizontal line in the box while
the possible outputs are found below the line. Outputs from each tool can be
saved and/or used as in the input for the next tool. Selecting the box will
display the settings associated with the tool allowing you to preset parameters
to reuse each time the workflow is run.

### Extract Workflow from Current History
1. Select the Gear Icon from the top of the history pane.
2. Select `Extract Workflow` from the menu.
3. In Workflow name enter "QC and Cleanup".
4. Select `Create Workflow`.

### View and Edit the Workflow
1. Select Workflow from the top bar.
2. Select the workflow that you just created and select Edit from the drop down
   menu.
3. The initial view of the workflow may be very messy. You can drag the boxes
   around on the screen to make the workflow easier to interpret. You can also
   move the blue box in the bottom right corner to view other sections of the
   workflow.
4. The workflow will follow the same logic as the history you created it from.
   Can you trace the steps you took for each initial FASTQ file though the QC
   and clean up process?
5. Select a FastQC:Read QC box which will open the Details Pane on the right.
   Is this for the Left or Right reads?
6. Select the Input dataset box that is attached (connected arrows) to the
   FastQC:Read QC box you just viewed.
7. Label the Input dataset either Left or Right to correspond with the
   information from the FastQC:Read QC box you just viewed.
8. Do the same for the other Input dataset.
9. The next time you need to run QC and clean up FASTQ data you might need to
   use different adapter sequences. Selecting the Trimmomatic box to view the
   options in the Details Pane.
10. In the Details Pane for Trimmomatic change the adapter sequence to
   `Set at runtime` using the small arrow next to `fastaWithAdapters`.
11. Select the Gear Icon then Save from the menu.

### Running a Workflow
1. Select Analyze Data from the top bar to return to the main Galaxy screen.
2. Create a new history by selecting the Gear Icon then Create New from the menu.
3. Name the history `Workflow Test`
4. Import `Tutorial file workflow R1.fastq` and `Tutorial file workflow R2.fastq`
   into the current `Workflow Test` history from the data library. Don't forget
   to set the file attributes.
5. Select Workflow from the top bar to display your saved workflow from the data
   library.
6. Select the QC and Cleanup workflow you just created then select Run from the
   drop down menu.
7. Set the input datasets and the adapter using the drop down menus.
8. Scroll to the bottom of the main view and select Run workflow
9. Select Analyze Data in the top bar to return to the main Galaxy view.
10. You will be able to watch the progress of the workflow in the History Pane

## Part 7: Sharing Workflows and Histories
It is possible to share workflow and histories with other Galaxy users. This
allows you to share data, results and methods with collaborators or anyone who
might want to recreate your methodology. Galaxy histories and workflows can be
shared via a link or they can be saved as stand alone files that can then be
uploaded to any Galaxy instance.

### Share a History
1. To share your current history select the Gear Icon then Share or Publish
2. To share the history though a web link select Make History Accessible via
   Link. You can share this link with anyone who has access to Galaxy at MSI
   allowing them to view the history and the data in it.
3. Make History Accessible and Publish will also create a link to the history
   but it will also publish the history making it public to anyone with access
   to Galaxy at MSI under the Shared Data tab.

### Share a Workflow
1. When you select a workflow from the list one of the options is Share or
   Publish.
2. To share the workflow though a web link select Make Workflow Accessible via
   Link. You can share this link with anyone who has access to Galaxy at MSI
   allowing them to view and use the workflow.
3. Make Workflow Accessible and Publish also creates a link to the workflow but
   it will also publish the workflow making it public to anyone with access to
   Galaxy at MSI under the Shared Data tab.
4. You can also download a workflow to be imported into another Galaxy instance
   or to be archived by selecting Download or Export.

## Part 8: Cleaning up Histories and Deleting Data from Galaxy
Galaxy is a shared resource so the amount of data you and your group can have in
Galaxy is limited. We estimate that even a relatively simple RNA-seq analysis
will use 4-5 times the storage of the raw sequencing files. Many of these files
are intermediate and can be discarded once the analysis is complete. Also, since
Galaxy allows you to create workflows it is easy to recreate intermediate files
if they are needed later. It is good practice to extract workflows from
histories then discard the histories once you have completed the analyses.

Your groups current Galaxy usage is displayed in the top bar on the far right.
You can view the size of your different Galaxy histories when you view you saved
Galaxy histories using the Gear Icon and below the name of your current history.
If you delete a Galaxy history before you extract a workflow you will not be
able to do so later.

### Deleting Intermediate Files and Histories from Galaxy
1. Select Analyze Data in the top bar to get to the main Galaxy view.
2. Select the Gear Icon and then Saved Histories from the menu.
3. Select the history you created when you tested your workflow then select
   Switch to open the history in the Galaxy History Pane.
4. To delete specific pieces of data from a Galaxy history you can select the X.
5. Notice that when the data set is deleted that the size of the history does
   not change. This is because Galaxy has a recycling bin type function.
6. To permanently delete a dataset first unhide the hidden datasets by selecting
   the Gear Icon then Include Deleted Datasets.
7. Select the here link displayed in the history pane for the data you would
   like to delete. This will actually reduce the size of the history
