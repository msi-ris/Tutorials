---
layout: default
title: Illumina Data QC Tutorial at the Command Line
permalink: /illumina_cmd/
exclude: false
updated: 2014-09-16
delivered: 2018-06-13
---

# Illumina Data Quality Control at the Command Line
*Last Updated: {{page.updated}}*

*Last Delivered: {{page.delivered}}*

This tutorial is also available as a [PDF]({{"/materials/illumina_qc_cmd/Illumina_QC_Cmd_Current.pdf" | prepend: site.baseurl }}).

## Part 0: Introduction
### Quick UNIX Introduction
UNIX is an operating system like OSX or Windows. The interface between you and
the UNIX OS is called "the shell." There are a few flavors of shell but the MSI
standard is bash. The shell is what you see, your environment, when you open
PuTTY or Terminal.

UNIX acts like any other operating system and allows you to
store and create files then use them during processes (e.g. running a program).
Unlike OSX or Windows UNIX is not visual and has to be navigated using simple
commands though the shell.

### Note About Remote Computing
The goal of this tutorial is to introduce you to using MSI computational
resources. This means you are going to use your computer to talk to other
computers and tell those computers what to do. This also means that you have to
make sure you know how to navigate a different computational environment (UNIX)
and the data you need is somewhere MSI systems can access it (i.e. not on your
personal computer).

### Brief Outline of MSI Systems
- **Login Node**

  When you access MSI systems the login node is the first system that you
  connect to. The login node is only there to direct you to other MSI systems.

- **Lab System**

  A group of computers that can be accessed using the isub command. This is an
  older system with less computational power behind it.
  <https://www.msi.umn.edu/labs/pbs>.
