---
layout: default
title: Illumina Data QC Tutorial
permalink: /illumina/
exclude: false
updated: 2018-06-13
---

# Illumina Data Quality Control
Last Updated: {{page.updated}}

This tutorial is also available as a PDF.

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
    – Galaxy layout
    – Loading files into current Galaxy history
    – Creating a workflow
    – Sharing histories and workflows
    – Where to get more information
- Basic processing and quality control on Illumina sequencing data
    – Evaluating read quality
    – Adapter removal
    – Low quality read removal
    – Read trimming
