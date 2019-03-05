---
layout: default
title: Intro to RNASeq (Lecture)
permalink: /rnaseq_lec/
exclude: false
updated: NA
delivered: NA
---

<div id="navdiv" markdown="1">

## Table of Contents
- [Introduction](#0)

</div>

# This Tutorial Is Under Construction!

# <a name="top"></a> {{page.title}}
*Last Updated: {{page.updated}}*  
*Last Delivered: {{page.delivered}}*

## <a name="0"></a>Part 0: Introduction
### <a name="0.1"></a>0.1 Formatting in This Document
Throughout this tutorial, there will be formatting cues to highlight various
pieces of information.

<div class="warn" markdown="1">

This is a warning. Common pitfalls, cautionary inforation, and important points
to consider will appear like this.

</div>

```
File contents or literal values will appear like this. We will use this format
to illustrate file formats.
```

[Return to top](#top)
### <a name="0.2"></a> 0.2: Goals
- Learn how RNAseq techniques can be applied to your research program
- Become familiar with several core file formats of genomics
- Learn the basic steps of RNASeq data analysis, including read mapping,
  expression counts, and testing for differential expression

[Return to top](#top)
### <a name="0.3"></a> 0.3: Scope of the Tutorial
This tutorial has a lecture format and will cover the principles and concepts
that underlie RNAseq experimental design and data analysis. This tutorial will
not cover the mechanistic details of how to analyze RNAseq data - that will
be covered in the companion hands-on tutorial.

[Return to top](#top)

## <a name="1"></a> Part 1: RNASeq Overview
### <a name="1.1"></a> 1.1: What is RNASeq?
RNAseq is a technique that uses high-throughput sequencing technologies to
assay the *transcriptome*, or the transcribed portions of a genome. Which parts
of the genome are transcribed depends on many factors, including biotic or
abiotic stressors, organ or tissue type, developmental stage, and genotype of
the samples. RNAseq data can be used to assemble transcript sequences, detect
alternative splicing events, or test for differential expression across a set
of conditions. Because RNAseq data is sequence-based, it can even be used to
identify genetic polymorphisms, with the caveat that they will only be found
in highly-expressed regions of the genome.

Relatively new sample handling library preparation protocols allow researchers
to tag sequencing reads as originating from the RNA of a single cell
(single-cell RNAseq, scRNAseq). This allows researchers to study expression at
very fine-grained resolution. Single-cell RNAseq requires special
considerations, both in terms of analysis and wet-bench molecular techniques,
and we will not cover it in high detail here.

"Typical" RNAseq protocols involve extraction of RNA from groups of cells that
are homogenized, or bulked. It is broadly labeled as *bulk RNAseq* to
distinguish it from scRNAseq. Most of the material we will cover in this
tutorial will cover either general experimental design or bulk RNAseq handling.

### <a name="1.2"></a> 1.2: RNAseq vs. Other Genomics Technologies
Because RNAseq targets transcribed, or expressed, parts of the genome, its
most direct comparisons are to other expression-focused genomics technologies.

In comparison to expression microarrays, RNAseq has higher dynamic range and
does not depend on a designed assay, which means that it potentially has higher
sensitivity and is easier to apply to systems which are not genomics model
systems. However, the analytical framework in place for analyzing microarray
data is more mature and standardized, meaning that RNAseq studies may have lower
repeatability than those based on microarrays. RNAseq analysis is becoming more
standardized, however, and a strong grounding in experimental design can allow
researchers to account for bias and experimental error in their analysis. RNAseq
also allows single-nucleotide resolution, which means that the data can be used
to discover variants as well as expression differences.

In comparison to EST (expressed sequence tag) sequencing, RNAseq is much faster
and cheaper per nucleotide of data. EST sequencing does not rely on a reference
genome sequence, but is much lower throughput. EST sequencing also does not
yield any information about the *relative* expression levels of various
transcripts, so it precludes differential expression analysis.

RNAseq is relatively affordable, has high sensitivity to detect expression
differences, can be used to detect isoforms and genetic variants, but has high
noise and can suffer from low repeatability. However, like scientific research
in general, careful experimental design can mitigate the repeatability problems.

[Return to top](#top)

## <a name="2"></a> Part 2: RNASeq Experimental Considerations
Designing an RNAseq experiment draws upon the same principals of general
experimental design. In principle, the expression values of the genes that
are assayed by RNAseq technologies are phenotypes that are being measured by the
researcher. Randomization, replication, and blocking are all important
components of an RNAseq experiment, and should be included where logistically
(and financially) possible.

### <a name="2.1"></a> 2.1: Types of RNASeq Experiments
The exact design of an RNAseq experiment will depend on the researcher's goals.
For example, for transcriptome assembly, a researcher may want to sample
several replicates each from multiple tissues of an organism and sequence them
to high depth. For differential gene expression analysis, a design that samples
a single tissue from multiple biological replicates within each treatment is
more appropriate.

#### Pairwise Two-Group Comparison
This is a very simple design that is used to test the effect of a single factor,
such as a drug compound or a single stress condition.

![Pairwise Experiment]({{ "/graphics/rnaseq_lec/pairwise_exp.png" | prepend: site.baseurl }})

In this example, the black samples represent the control condition and the
red samples represent the treatment condition. There are several *biological*
replicates per condition, which usually means different individuals of the same
strain or genotype that were subjected to the same treatment.

#### Factorial Design
This design is slightly more complicated and is used to test for the effect of
interaction between two experimental variables. For example, the effects of
both heat stress and salt stress on plant gene expression.

![Factorial Experiment]({{ "/graphics/rnaseq_lec/factorial_exp.png" | prepend: site.baseurl }})

In this example, the black circles are the control group, having received
neither heat nor salt treatment. The blue circles have been subjected to the
salt treatment, but not the heat treatment. The red circles were treated with
heat, but not salt. The violet circles are samples that were treated with both
heat and salt.

This design allows the researcher to isolate both the effects of heat stress
and salt stress on gene expression, and includes a group for testing the
interaction between heat and salt.

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

[Return to top](#top)
## Part 9: Feedback
This tutorial document was prepared by Thomas Kono, in the RIS group at MSI.
Please send feedback and comments to konox006 [at] umn.edu. You may also send
tutorial delivery feedback to that address.

[Return to top](#top)
