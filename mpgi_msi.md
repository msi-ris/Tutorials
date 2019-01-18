---
layout: default
title: MPGI-MSI Workshop
permalink: /mpgi_msi/
exclude: false
updated: 2019-01-18
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
- [System status](https://www.msi.umn.edu/systemstatus); this page is in the process of being rebuilt. This link will update to reflect
the new status page when it launches.
- [Queue usage](https://www.msi.umn.edu/qstat), dynamically displays queue loads


## <a name="2"></a> Module 2: Introduction to Genomics Techniques and Terminology
### Slide Deck
You can view the slide deck [here](https://drive.google.com/open?id=1FrTF8Kp6j3sCJq0bgkGvgqLqYQ-5cj9eLmxV3IiHdAs).

## <a name="3"></a> Module 3: Introduction to PBS and Job Submission
### Slide Deck
You can view the slide deck [here](https://drive.google.com/open?id=14-QitVjoM7uOo2QPNqWPD7TD2vPP3YzR5FicdtimCho).

## <a name="4"></a> Module 4: Parallelization with GNU Parallel and Task Arrays
### Slide Deck
You can view the slide deck [here](https://drive.google.com/open?id=1eD-qoXzRS2YI_MqI6OzVpNwA_IXHx07WXd_VM-6mQM8).

## Feedback
This tutorial document was prepared by Thomas Kono, in the RIS group at MSI.
Please send feedback and comments to konox006 [at] umn.edu. You may also send
tutorial delivery feedback to that address.

[Return to top](#top)
