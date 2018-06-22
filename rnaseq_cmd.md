---
layout: default
title: RNASeq Analysis With the Command Line
permalink: /rnaseq_cmd/
exclude: false
updated: 2018-06-22
delivered: NA
---

# {{page.title}}
*Last Updated: {{page.updated}}*

*Last Delivered: {{page.delivered}}*


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

  Setup: <https://www.msi.umn.edu/support/faq/how-do-i-configure-putty-connect-msi-unix-systems>  
  Unlike Mac/OSX windows does not have a native UNIX terminal. PuTTY is a
  terminal emulator for Windows and will allow you to interact with MSI systems
  using UNIX commands.

## Part 2: Accessing MSI Systems and Getting Tutorial Data
On OSX or Linux, open your terminal emulator, and connect to the MSI login nodes
with `ssh username@login.msi.umn.edu`.

On Windows, open PuTTY, and follow the instructions at
[this link](https://www.msi.umn.edu/support/faq/how-do-i-configure-putty-connect-msi-unix-systems).

```
konox006-MacBookAir:~ konox006$ ssh konox006@login.msi.umn.edu
konox006@login.msi.umn.edu's password: *************
```

If everything checks out, you will see the MSI message-of-the-day (MOTD), which
confirms that you have successfully logged in.

```
Last login: Fri Jun 15 11:00:20 2018 from dhcp06.msi.umn.edu
-------------------------------------------------------------------------------
              University of Minnesota Supercomputing Institute
-------------------------------------------------------------------------------
For assistance please contact us at
https://www.msi.umn.edu/support/request,
help@msi.umn.edu, or 612-626-0802.
-------------------------------------------------------------------------------
This is a login host. Please avoid running resource-intensive tasks on
this machine.

MSI provides "Quick Start Guides"
  https://www.msi.umn.edu/quick-start-guides

For interactive workloads:
  https://www.msi.umn.edu/content/interactive-hpc

For batch workloads:
  https://www.msi.umn.edu/content/hpc

-------------------------------------------------------------------------------
Files in /tmp and /scratch.global are automatically removed
periodically. Please ensure finished work is copied to your home directory.
-------------------------------------------------------------------------------
konox006@login03 [~] %
```

The final line is your **prompt**, which is where you enter commands. It lists
your username, host, and current directory. As the `@loginXX` after your
username in the ternimal indicates, you are now connected to the login node.
While you can move around the filesystem and use UNIX commands while connected
to the login node you won't be able to use any of the software installed on MSI
systems. In order to gain access to the installed software you will need to
connect to one of the HPC systems. We are going to connect to the lab system
though a command called `isub`.

Type `isub -m 8gb -w 4:00:00` and press enter to request an interactive session
on the Lab HPC system. This command will request 8gb of memory for four hours.
If you would like to see all options availale for `isub`, type `isub -h` and 
press enter.

```
konox006@login03 [~] % isub -m 8gb -w 4:00:00
qsub: waiting for job 217341.nokomis0015.msi.umn.edu to start
qsub: job 217341.nokomis0015.msi.umn.edu ready
konox006@labq59 [~] %
```

The prompt updates to show us that we are logged into a lab node now. Let's see
where we are in the file system. Use the `pwd` command to do this. When you log
into the system for the first time you are automatically taken to your home
directory. Your home directory will always have the format of
`/home/yourGroup/yourMSIaccount`.

```
konox006@labq59 [~] % pwd
/home/msistaff/konox006
konox006@labq59 [~] %
```

You can list directory contents with the `ls` command:

```
konox006@labq59 [~] % ls
bin    Downloads  Music Public       Soft       Videos
Desktop    igv        Pictures  R      Templates
Documents  logs       Projects  smrtlinktunnel.sh  tmp
```

And you can use `cd` to move around the directory structure. The `..` directory
means to move "up" one level.

```
konox006@labq59 [~] % cd ..
konox006@labq59 [/home/msistaff] % ls
... (names removed) ...
```

The tutorial files are located in the `/home/msistaff/public/qcIllumina`
directory. Let's navigate there.

```
konox006@labq59 [~] % cd /home/msistaff/public/qcIllumina
konox006@labq59 [/home/msistaff/public/qcIllumina] % ls
fastqc_only.sh  Tutorial_file_R1.fastq  tutorial_trim.sh
trim_loop.sh  Tutorial_file_R2.fastq
tutorial  tutorial_trim2.sh
```

This directory listing format isn't very useful. We can use the `-l` option to
get a long listing of the files, and the `-h` option to show us the sizes, in
human-readable formats. The long format contains (among other information), a
permission string, an owner name, a group name, size, modification date, and the
filename.

```
konox006@labq59 [/home/msistaff/public/qcIllumina] % ls -lh
total 277M
-rwxrwxr--. 1 ljmills  msistaff  355 Sep 18  2014 fastqc_only.sh
-rw-r-----. 1 ljmills  msistaff 1.2K Apr  6  2017 trim_loop.sh
drwxr-sr-x. 2 fallo002 msistaff 4.0K Jun 28  2017 tutorial
-rwxrwxrwx. 1 ljmills  msistaff 123M Sep 18  2014 Tutorial_file_R1.fastq
-rwxrwxrwx. 1 ljmills  msistaff 123M Apr  8  2017 Tutorial_file_R2.fastq
-rwxrwxr--. 1 ljmills  msistaff  684 Nov 19  2015 tutorial_trim2.sh
-rwxrwxr--. 1 ljmills  msistaff  558 Jul 17  2015 tutorial_trim.sh
```

Go back to your home directory. There are multiple ways to do this. You can
run `cd` without any arguments, use the `~` shorthand for your home directory,
or type out the full path. The following are equiavalent:

```
konox006@labq59 [/home/msistaff/public/qcIllumina] % cd
konox006@labq59 [/home/msistaff/public/qcIllumina] % cd ~
konox006@labq59 [/home/msistaff/public/qcIllumina] % cd /home/msistaff/konox006
```

Now that you're in the home directory, you can make a new directory for the
tutorial and copy the example data into it. The files you will copy are called
`Tutorial_file_R1.fastq` and `Tutorial_file_R2.fastq`.

```
konox006@labq59 [~] % mkdir tutorial
konox006@labq59 [~] % cd tutorial
konox006@labq59 [~/tutorial] % cp /home/msistaff/public/qcIllumina/Tutorial_file_R1.fastq .
konox006@labq59 [~/tutorial] % cp /home/msistaff/public/qcIllumina/Tutorial_file_R2.fastq .
```

Note the trailing `.` at the end of the `cp` command - it is a shorthand to
refer to the current working directory. Now is also a good time to point out
that you don't have to remember those long directory paths - you can type the
first few characters, then press the `Tab` key to have the shell auto-fill in
the rest of the filename.


## Part 3: Running FastQC
Next, we will use the `fastqc` tool to assess the quality of the sequences in
the FASTQ files that we just copied. To get access to the `fastqc` program, we
have to load the *software module*. This can be done with the `module` command.

```
konox006@labq59 [~/tutorial] % module load fastqc/0.11.7
```

Then, you can analyze the FASTQ file. This may take a minute or two, and you
will see progress output written to the terminal while `fastqc` works.

```
konox006@labq59 [~/tutorial] % fastqc Tutorial_file_R1.fastq
Started analysis of Tutorial_file_R1.fastq
Approx 5% complete for Tutorial_file_R1.fastq
Approx 10% complete for Tutorial_file_R1.fastq
Approx 15% complete for Tutorial_file_R1.fastq
Approx 20% complete for Tutorial_file_R1.fastq
Approx 25% complete for Tutorial_file_R1.fastq
...
Analysis complete for Tutorial_file_R1.fastq
konox006@labq59 [~/tutorial] % ls
Tutorial_file_R1.fastq        Tutorial_file_R1_fastqc.zip
Tutorial_file_R1_fastqc.html  Tutorial_file_R2.fastq
```

`fastqc` created two new files: an HTML file and a ZIP file. The HTML file
contains the quality report of the sequences, and the ZIP file contains the
raw data used to generate the HTML report. Use FileZilla to copy the HTML file
to your local machine to view it in a web browser.

## Part 4: Cleaning Reads
We will use `trimmomatic` to clean the low quality bases and adapter
contamination from the reads. Like with `fastqc`, you access it with the
`module` command:

```
konox006@labq59 [~/tutorial] % module load trimmomatic
konox006@labq59 [~/tutorial] % _JAVA_OPTIONS="-Xmx4g" java -jar \
> ${TRIMMOMATIC}/trimmomatic.jar PE -phred33 \
> Tutorial_file_R1.fastq Tutorial_file_R2.fastq \
> R1.PE.fastq R1.SE.fastq R2.PE.fastq R2.SE.fastq \
> ILLUMINACLIP:${TRIMMOMATIC}/adapters/TruSeq2-PE.fa:2:30:10:2:true \
> LEADING:3 TRAILING:3 SLIDINGWINDOW:4:15 MINLEN:30
```

Note that the `\` and the new lines are merely for readability. You can skip
those and enter all of the command on a single line. This will also take a
few minutes to run.

Check the current directory contents. What got created? Run `fastqc` on the new
PE FASTQ files. Use the `*` (wildcard character) to cut down on the number of
commands that you have to type:

```
konox006@labq59 [~/tutorial] % ls
hs_err_pid24802.log  R2.PE.fastq       Tutorial_file_R1_fastqc.html
R1.PE.fastq      R2.SE.fastq       Tutorial_file_R1_fastqc.zip
R1.SE.fastq      Tutorial_file_R1.fastq  Tutorial_file_R2.fastq
konox006@labq59 [~/tutorial] % fastqc *PE.fastq
...
```

### A Note About Scripting
While interactive sessions are useful for running quick analyses, there are some
important disadvantages to keep in mind.

1. You have to build the commands properly each time.
2. You may have to rerun the same commands with slight modifications if there
   are many files (or build a loop interactively).
3. You have to wait for each command to finish before moving on.
4. **You don't have a record of your work.**

The best way to address each of these issues is to submit jobs to the queue
as shell scripts. Your script will still need to have properly formatted
commands, but you will have a file that lists exactly the parameters that you
specified and the input and output files that you used.

We have provided an example script that runs `trimmomatic` to get you started.
Copy `/home/msistaff/public/qcIllumina/tutorial_trim.sh` into the tutorial
directory. Open it with Komodo Edit with `File` > `Open` > `Remote File`. Enter
your MSI login credentials, then navigate to the tutorial script.

```
konox006@labq59 [~/tutorial] % cp /home/msistaff/public/qcIllumina/tutorial_trim.sh .
```

The contents of the script are shown below:

```bash
#!/bin/bash -l
#PBS -l nodes=1:ppn=6,mem=15GB,walltime=1:00:00
#PBS -m ae
#PBS -M youremail
#PBS -e trimmomatic.error
#PBS -o trimmomatic.out
#PBS -N tutorial_trim

module load trimmomatic
module load fastqc

cd /home/msistaff/konox006/tutorial
fastqc Tutorial_file_R*.fastq

java -jar $TRIMMOMATIC/trimmomatic.jar PE \
    -phred33 Tutorial_file_R1.fastq Tutorial_file_R2.fastq \
    R1.PE.fastq R1.SE.fastq R2.PE.fastq R2.SE.fastq \
    ILLUMINACLIP:$TRIMMOMATIC/adapters/TruSeq2-PE.fa:2:30:10:2:true \
    LEADING:3 TRAILING:3 SLIDINGWINDOW:4:15 MINLEN:30
fastqc *.PE.fastq
```

The first line is a special line that tells the operating system to use the
`bash` shell interpreter to run the script. When we enter commands
interactively, we are doing it through the `bash` shell. The next four lines
that start with `#PBS` are special lines that declare the compute resources that
the job will require. Fill in your email address after the `-M`. For a more
complete description of the information in the `#PBS` lines, see [this link](https://www.msi.umn.edu/content/job-submission-and-scheduling-pbs-scripts). To see allowed limits for
computational resources, see [this link](https://www.msi.umn.edu/queues).

The next lines are all regular `bash` scripting lines. Note that these are
identical to the commands that we entered interactively in the previous steps.
Be sure to edit the script so that the directories given are for your MSI
account - you will get permissions errors if you try to work with another
person's files.

Once you edit and save the script, submit it to the job queue with the `qsub`
command:

```
konox006@labq59 [~/tutorial] % qsub tutorial_trim.sh
675304.nokomis0015.msi.umn.edu
```

The text that writes to the terminal is the ID of the job, and confirms that
your job has been sucessfully submitted. If you get a different message, it
means there was some error in your job request, usually in the `#PBS` lines.
Check your spelling and limits allowed in the queues and try again. You will
get email notifications when your job starts and when your job exits.

## Part 5: Automating FastQC and Trimmomatic With Scripts
This script is good - it runs `fastqc` on the raw reads, then cleans them with
`trimmomatic`, then runs `fastqc` on the cleaned output. But it's not very
flexible and would require a lot of modifications or a different dataset. In
this section, we will look at some more advance scripts that will both teach
cool tricks and give you a more general script that can be used across multiple
projects with minimal rewriting.

Copy the `fastqc_only.sh` script out of the tutorial directory. Open it with
Komodo Edit like the first script.

```
konox006@labq59 [~/tutorial] % cp /home/msistaff/public/qcIllumina/fastqc_only.sh .
```

The contents of this script are shown below:

```bash
#!/bin/bash -l
#PBS -l nodes=1:ppn=4,mem=15GB,walltime=1:00:00
#PBS -m ae
#PBS -e trimmomatic.error
#PBS -o trimmomatic.out
#PBS -N trimmomatic

module load trimmomatic
module load fastqc

cd /home/msistaff/konox006/tutorial

FASTQ="/home/msistaff/konox006/tutorial"

for F in Tutorial_file_R1.fastq Tutorial_file_R2.fastq
do
    fastqc -o $FASTQ $F
done
```

This script looks simple, but it has a few new concepts. The first is a
*variable*. Both the `$FASTQ` and `$F` are variables. When you set the value
of a variable (or "declare" or "initialize" it), you do not need the `$` sigil
in front. There cannot be any spaces between the variable name, the `=`
operator, and the value when you want to assign values. When you want to use the
value (or "reference" it), you need to use the `$` sigil. Naming variables in
ALLCAPS is not necessary, but it is a convention to do so, to make them easier
to see.

The next concept is the `for` loop. This construct lets you run a sequence of
commands without having to type each one individually. This is very useful when
you want to run the same command on many files. The general structure of a `for`
loop is as follows:

```bash
for VAR in LIST
do
    command $VAR
done
```

This will take a list of things (files, numbers, etc.), and sequentially run
`command` on each item. The current value of the loop is stored in the variable
`$VAR`. The `do` and `done` keywords are required for `for` loops. In our
example, the `LIST` has two elements, `Tutorial_file_R1.fastq` and
`Tutorial_file_R2.fastq`, and we will run `fastqc` on each of them.

You can also build the list dynamically, using the output of another command.
This next script uses *command substitution* to do this. The command to be
run is between the `$()` characters. In this case, we run `ls *.fastq`, which
returns a list of files ending in `.fastq`, and use that as the list for
iteration. Command substution looks somewhat like a variable reference, but they
are very different:

```bash
#!/bin/bash -l
#PBS -l nodes=1:ppn=4,mem=15GB,walltime=1:00:00
#PBS -m ae
#PBS -e trimmomatic.error
#PBS -o trimmomatic.out
#PBS -N trimmomatic

module load trimmomatic
module load fastqc

cd /home/msistaff/konox006/tutorial

FASTQ="/home/msistaff/konox006/tutorial"

for F in $(ls *.fastq)
do
    fastqc -o $FASTQ $F
done
```

Let's combine the FastQC calls with the Trimmomatic calls. Because Trimmomatic
requires two input files, you need to be a bit more sophisticated with data
in the `for` loop. Copy the tutorial FASTQ files back into your directory, and
give them new names. We will do this just to illustrate how to iterate over
multiple sets of FASTQ files.

```
konox006@labq59 [~/tutorial] % cp /home/msistaff/public/qcIllumina/Tutorial_file_R1.fastq new_tutorial_file_R1.fastq
konox006@labq59 [~/tutorial] % cp /home/msistaff/public/qcIllumina/Tutorial_file_R2.fastq new_tutorial_file_R2.fastq
```

Copy the `tutorial_trim2.sh` file from the tutorial directory and open it Komodo
Edit. The contents of the script are below:

```bash
#!/bin/bash -l
#PBS -l nodes=1:ppn=4,mem=15GB,walltime=1:00:00
#PBS -m ae
#PBS -e trimmomatic.error
#PBS -o trimmomatic.out
#PBS -N trimmomatic

module load trimmomatic
module load fastqc

cd /home/msistaff/konox006/tutorial

FASTQ="/home/msistaff/konox006/tutorial"

for F in "Tutorial_file_R1.fastq Tutorial_file_R2.fastq" "new_tutorial_file_R1.fastq new_tutorial_file_R2.fastq"
do
set -- $F
R1="$FASTQ/$1"
R2="$FASTQ/$2"
fastqc -f fastq $R1 $R2 -o $FASTQ
java -jar $TRIMMOMATIC/trimmomatic.jar PE \
    -phred33 $R1 $R2 ${1}.PE ${1}.SE ${2}.PE ${2}.SE\
    ILLUMINACLIP:$TRIMMOMATIC/adapters/TruSeq2-PE.fa:2:30:10 \
    LEADING:3 TRAILING:3 SLIDINGWINDOW:4:20 MINLEN:36
fastqc ${1}.PE ${2}.PE
done
```

Notice how the pairs of FASTQ files are enclosed in double quotes (`""`). This
is to keep them associated during iteration through the `for` loop. Then, in
the set of commands to run, we split them into the `$1` and `$2` variables with
`set -- $F`. Also notices how some of the variables also have curly braces
(`{}`) around them. This is to clearly define where the name of the variable
starts and stops. `bash` will read `$1.PE` as a variable named `1.PE`, which we
have not defined. In order to have the value of `$1` be substituted with a `.PE`
at the end, we have to enclose the `1` in curly braces: `${1}.PE`. In general,
I always enclose my variable names in curly braces because it is the safest
way to reference them, even if it is a bit of extra typing.
