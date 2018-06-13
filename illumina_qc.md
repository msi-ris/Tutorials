---
layout: default
title: Illumina Data QC Tutorial With Galaxy
permalink: /illumina/
exclude: false
updated: 2018-06-13
delivered: 2018-06-13
---

# Illumina Data Quality Control With Galaxy
*Last Updated: {{page.updated}}*

*Last Delivered: {{page.delivered}}*

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
