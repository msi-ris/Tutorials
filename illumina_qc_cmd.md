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
### Goals
- Log into MSI and connect to the Lab system.
- Learn some basic UNIX commands.
- Learn how to load software using the module system
- Learn how to run FastQC and Trimmomatic at the command line
- Learn how to write a bash script to automate running FastQC and Trimmomatic
- Learn how to convert the bash script to work with PBS to submit jobs to the
  queue.

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

- **Itasca System**

  This is the newer, faster, fancier computer system. Your group will need to
  have SUs to use this system. SUs are free; Itasca is awesome; sign up for SUs!
  <https://www.msi.umn.edu/resources/job-queues>

- **PBS queuing**

  PBS is a queuing program that takes care of reserving and ordering jobs to be
  run on the different systems. PBS takes special commands that allow you to ask
  for a specific amount of time and computational power. Once you submit your
  job (i.e. program) it will get in line (in the queue) and will run once room
  opens up.

- **Storage**

  Each group is allotted some amount of storage the default amount is 100GB but
  your group can request as much storage as you can justify.

## Part 1: Setup
### Software Requirements
- **Komodo Edit**

  <http://www.activestate.com/komodo-edit/downloads>

  This editor will allow you to write and edit scripts and save them to your
  MSI space. By connecting Komodo Edit to MSI you can write, saved and edit
  documents and scripts that are saved in MSI space. Komodo Edit is an
  alternative to learning a true editor such as VI, Nano or Emacs. You should
  still learn how to use an editor because you might not always have Komodo and
  the editors can do much fancier tricks.

  `Preferences->Servers-> + button` Server type: SFTP  
  Name: MSI  
  Hostname: `login.msi.umn.edu`  
  Username: `<your MSI username>`  
  Password: `<your MSI password>`

- **FileZilla Client**

  <https://filezilla-project.org/>

  Setup: <https://www.msi.umn.edu/support/filezilla>  
  This SFTP/FTP client will help you move files into and out of your MSI space.  

- **PuTTY** (*Windows Users Only*)

  <http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html>

  Setup: <https://www.msi.umn.edu/support/putty>  
  Unlike Mac/OSX windows does not have a native UNIX terminal. PuTTY is a
  terminal emulator for Windows and will allow you to interact with MSI systems
  using UNIX commands.
