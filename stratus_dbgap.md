---
layout: default
title: Interacting with dbGaP Data on Stratus
permalink: /stratus_dbgap/
exclude: false
updated: 2022-01-28
delivered: NA
---

<!-- Some daft javascript to open all <details> on print. -->
<script type="text/javascript">
function open_det(e) {
  e.open = true;
}
function close_det(e) {
  e.open= false;
}
function openall() {
  var det_elems = document.getElementsByTagName('details');
  var arr_det_elems = Array.prototype.slice.call(det_elems);
  arr_det_elems.forEach(open_det);
}
function closeall() {
  var det_elems = document.getElementsByTagName('details');
  var arr_det_elems = Array.prototype.slice.call(det_elems);
  arr_det_elems.forEach(close_det);
}
</script>

<div id="navdiv" markdown="1">

## Table of Contents
- [Introduction](#0)
    - [Formatting in This Document](#0.1)
    - [Goals](#0.2)
    - [Scope of the Tutorial](#0.3)
    - [Required Software](#0.4)
- [Getting Authorized](#1)
    + [As a PI](#1.1)
    + [As an Analyst](#1.2)
- [What is Stratus?](#2)
    + [Alternatives](#2.1)
- [Accessing Stratus](#3)
- [Creating a VM](#4)
    + [Security Groups](#4.1)
    + [SSH Keys](#4.2)
    + [Virtual Machine](#4.3)
    + [Data Volume](#4.4)
- [Accessing a Running VM](#5)
    + [Mountain a Data Volume](#5.1)
- [Downloading dbGaP Data](#6)
    + [AsperaConnect](#6.1)
    + [SRAToolKit](#6.2)
    + [Downloader Key](#6.3)
    + [SRA dbGaP Data](#6.4)
    + [Non-SRA dbGaP Data](#6.5)
    + [Decrypting dbGaP Data](#6.6)
- [Installing Software](#7)
    + [With "yum"](#7.1)
    + [With RIS Conda Files](#7.2)
- [Adding New Users and Groups](#8)
- [Moving Data Out of Stratus](#9)
- [Advanced Tips and Tricks](#10)
    + [Making a Swapfile](#10.1)
    + [Managing Disk Input/Output](#10.2)

</div>

# <a name="top"></a> {{page.title}}
*Last Updated: {{page.updated}}*  
*Last Delivered: {{page.delivered}}*

<a onclick="javascript:openall()" href="#">Expand all details</a>  
<a onclick="javascript:closeall()" href="#">Collapse all details</a>

## <a name="0"></a>Part 0: Introduction
This tutorial is written to familiarize you with the policies regarding protected data on MSI systems through our Stratus service. The format for this tutorial is a guided workshop that will show you how to access Stratus and show you how to install a software suite for your analysis.

<div class="warn" markdown="1">

Stratus is a paid service from MSI. For current prices and resource allocations for Stratus subscriptions, please see this link:  
<https://www.msi.umn.edu/stratus>

</div>


### <a name="0.1"></a> Part 0.1 Formatting in This Document
Throughout this tutorial, there will be formatting cues to highlight various pieces of information.

<div class="info" markdown="1">

This is just background information. There are no tutorial-related tasks in these boxes. Links to supporting material and further explanations of points we raise in the tutorial will appear like this.

</div>

<div class="warn" markdown="1">

This is a warning. Common pitfalls, cautionary information, and important points to consider will appear like this.

</div>

```
This is code, or a literal value that you must enter or select to run a part of the tutorial
```

This tutorial involves running commands on multiple machines (your local workstation, the Stratus login host, and the virtual machine). Commands that are to be run on your local machine will have `(local machine)` before them; commands that are to be run on the Stratus login host will have `(stratus-login)` before them; and commands that are to be run on the virtual machine have `(virtual machine)` before them.

<details markdown="1">
<summary>These boxes contain detailed information. Click on them to expand them</summary>

When you click on these boxes, you will get a detailed view into a supplemental topic. The information in these boxes will be useful for advanced usage outside of the tutorial. We encourage you to read these!

</details>

Commands that are to be entered on the command-line will begin with `%`. Do not re-enter the `%` character when you type the commands into your prompt.

[Return to top](#top)
### <a name="0.2"></a> Part 0.2: Goals
- Learn if your data are protected access
- Access the MSI Stratus service
- Create a new Stratus virtual machine and launch it
- Log in to a running virtual machine
- Learn how to download dbGaP data into a virtual machine
- Learn how to install software on a virtual machine
- Learn how to manage access to your Stratus virtual machine

[Return to top](#top)
### <a name="0.3"></a> Part 0.3: Scope of the Tutorial

<div class="warn" markdown="1">

This tutorial requires that you have a device that is authorized for use with DUO two-factor authentication. If you do not have a device with DUO two-factor authentication, please see this guide from OIT:  
<https://it.umn.edu/technology/duo-two-factor-authentication>

</div>

This tutorial will cover the process of understanding if you or your PI will need to purchase a Stratus subscription, based on the nature of your dataset. We will also cover the steps required to authorize you to download and analyze data on Stratus.

This tutorial will assume that you have a basic understanding of interacting with UNIX or Linux systems via the command line. It will also assume that you have working knowledge of MSI systems and how they are organized.

This tutorial will not cover how to perform any specific analysis, nor will it cover the application process for accessing protected data in dbGaP.

[Return to top](#top)

### <a name="0.4"></a> Part 0.4: Required Software
The software required for this tutorial is listed below:

- Web browser
- Terminal emulator
- Secure shell (ssh)
- DUO authentication

## <a name="1"></a> Part 1: Getting Authorized to Use Protected Data
### <a name="1.1"></a> Part 1.1: As a PI
If you are a PI and would like to analyze NIH controlled-access data, follow the instructions at this link:  
<https://osp.od.nih.gov/scientific-sharing/requesting-access-to-controlled-access-data-maintained-in-nih-designated-data-repositories-e-g-dbgap/>

Be sure to read and follow the "Data Use Certification (DUC)" document and the "Genomic Data User Code of Conduct" - these describe how the data must be handled.

### <a name="1.2"></a> Part 1.2: As a Data Analyst
Before you begin to work on any protected data, you must be authorized to work with protected data. This involves contacting the university's Sponsored Projects Administration (SPA) office (<https://research.umn.edu/units/spa>) to register you with the NIH. The steps required are as follows:

1. Get an eRA Commons account. You can view instructions for how to get an account from UMN SPA by following this link:  
    <https://research.umn.edu/units/spa/proposals/electronic-grant-applications/register-nih-nsf-accounts>
    Get the "PI" status added to your new account. If you already have an account, you can skip this step.
2. If you already have an eRA Commons account, get a "PI" status added to it. If you already have "PI" status added, you can skip this step. To add PI status, contact eprops [at] umn.edu with your eRA Commons ID, and request the PI status.
3. Request that the principal for the dbGaP project authorize your eRA Commons ID as a "downloader" for the project. Section 4 of this NIH guide shows you how to do this:  
    <https://gdc.cancer.gov/access-data/obtaining-access-controlled-data>
4. Request the dbGaP project number, names, and UMN Internet IDs of any other researchers who will be accessing the dataset. The principal on the project should be able to supply these.
5. Send the information from step 4 to the current manager of the Stratus subscriptions.

## <a name="2"></a> Part 2: What is Stratus?
Stratus is a protected cloud computing environment hosted by MSI for processing NIH controlled-access data (that is, protected data). It uses the OpenStack platform to run Linux virtual machines within a sequestered environment. One of the key differences between Stratus and MSI's standard HPC facilities is that Stratus requires **multi-factor authentication** (DUO) to access.

Stratus has some nice advantages over MSI's main HPC services:

- You get *root* access to a Linux environment. This means that you have a lot of control over the compute environment in which your analysis takes place, such as installing whichever software packages you want.
- The resource allocation "floats" on top of a set of available hardware, which means that your compute time is not tied to any given node. Your compute environment remains available during server reboots and other downtime (e.g., monthly maintenance).
- You do not have to wait for jobs to be scheduled, so commands will run when you enter them on the command line.

However, there are some logistical concerns to be aware of when using Stratus:

- There is no `module` system on Stratus, and there is no software maintainer who will install packages for you. You must do this yourself.
- Stratus, by design, is isolated from Panasas (MSI primary storage). To transfer data between your virtual machine and Panasas, you must manually copy files between Stratus and primary storage through `scp` or Tier2 storage with `s3cmd`.
- It requires multi-factor authentication, so you will be using DUO a lot.
- Stratus requires a paid subscription.
- Stratus instances are smaller than the main HPC resources offered by MSI.

### <a name="2.1"></a> Part 2.1: Alternatives to Stratus
There are many providers for cloud computing, many of which are compliant with the NIH guidelines for protected data:

- Broad Institute FireCloud (<https://software.broadinstitute.org/firecloud/>)
- Institute SB Cancer Genomics Cloud (<https://isb-cgc.appspot.com/>)
- SevenBridges Cancer Genomics Cloud (<http://www.cancergenomicscloud.org/>)

Of course, there are benefits and drawbacks of these cloud services relative to Stratus!

Benefits:

- Cloud credits offered by NIH for free computations.
- Public IP addresses for non-UMN collaborators.
- Very, very, large scale compute resources.
- Tool kits, APIs, workflows that are supported by NIH.
- Charges based on what you use only.

Drawbacks:

- Unsupported by UMN and MSI.
- Equal time and effort investment to get system set up and workflows running.
- You are fully responsible for data security and access control.
- Beyond credit allocation, cost is much higher (~10X higher, as of 2018) per unit of compute time.

## <a name="3"></a> Part 3: Accessing Stratus
Once the principal has authorized you as a downloader and your Stratus subscription is active, you can access the Stratus dashboard through a web browser. Go to <https://stratus.msi.umn.edu/> and select `UMN OIT - Shibboleth (with DUO)` from the drop-down menu and click `Connect`. Log in with your UMN ID and password. You will need to further authenticate the login with DUO. Once you are authenticated, you will see a page that looks like the screenshot below.

![Openstack Dashboard]({{ "/graphics/stratus_dbgap/openstack_dashboard.png" | prepend: site.baseurl }})

The Openstack Dashboard lets you mange the virtual machines that you have made with your Stratus subscription. We will make only one virtual machine, but it is possible to have as many virtual machines as vCPUs that are in your subscription.

## <a name="4"></a> Part 4: Creating a Stratus Virtual Machine
We will now start the process of creating a new virtual machine. However, there are some steps that need to be taken before you create a new virtual machine for the first time. Note that the steps listed in parts [4.1](#4.1) and [4.2](#4.2) only need to be done once, no matter how many virtual machines you have in your Stratus project.

### <a name="4.1"></a> Part 4.1: Creating Security Groups
The first step that needs to be taken is to create a group that has permissions to use the secure shell (`ssh`) service. If you do not create this group and add yourself to it, then you will not be able to actually log in to your virtual machine once it is running. You will only have to create the SSH security group once; you can add this group to any and all of the virtual machines that you make with your Stratus project. To create the SSH group, click on the `Network` tab in the dashboard, then select `Security Groups`:

![Security Groups]({{ "/graphics/stratus_dbgap/security_group.png" | prepend: site.baseurl }})

The page will update to show you a listing of all of the current security groups that have been created for your Stratus subscription:

![Security Groups Listing]({{ "/graphics/stratus_dbgap/security_group_listing.png" | prepend: site.baseurl }})

Click the `Create Security Group` button. In the "Name" field of the pop-up box, enter `SSH`. In the "Description" box, enter `ssh` or some other descriptive text that will let you know what the group is for. Click on `Create Security Group`:

![Creating the SSH Group]({{ "/graphics/stratus_dbgap/ssh_group.png" | prepend: site.baseurl }})

The table should refresh, and a new row that contains the SSH group that you created should appear. Click on `Manage Rules` on the right side of the table row. The page will refresh and show you the rules that are active for the group. Click the `Add Rule` button on the top-right of the page:

![Adding rules to SSH group]({{ "/graphics/stratus_dbgap/add_ssh_rule.png" | prepend: site.baseurl }})

In the pop-up box, choose `SSH` for the "Rule" menu. Make sure the "Remote" field is `CIDR`, and make sure `0.0.0.0/0` is entered for the "CIDR" field. Click `Add`:

![Creating the ssh rule]({{ "/graphics/stratus_dbgap/ssh_rule.png" | prepend: site.baseurl }})

<div class="info" markdown="1">

If you would like to use remote desktop to access your virtual machine, then you need to create a security group for that as well. The process to do so is nearly identical to the process for SSH: instead of choosing `SSH` in the "Add Rule" dialog box, choose `RDP`. The values for "Remote" and "CIDR" are the same as for the `ssh` group.

</div>

### <a name="4.2"></a> Part 4.2: Generating SSH Keys
This section involves running commands on both your *local machine* and the remote *virtual machine*. Commands that are to be run on the local machine will have `(local machine)` before them, and commands that are to be run on the virtual machine will have `(virtual machine)` before them.

Now that you have made a group for `ssh` access, you must generate SSH keys for connecting from the protected Stratus gateway to a virtual machine. Like the SSH security group, the SSH keys need only be generated and registered once for your Stratus project. You can add the key you generate to as many virtual machines as you want within your project. The key is manged through the Openstack Dashboard. First, you must log in to the Status gateway.

<div class="warn" markdown="1">

`stratus-login` has very small user quotas (100MB). Users are not meant to transfer large data files to/from their virtual machines via the `stratus-login` host. Users should use Tier2 to move data between their virtual machines and primary MSI systems. See [Section 9](#9).

</div>

From your local machine, connect to `stratus-login.msi.umn.edu` with your UMN Internet ID and password. You will need to authenticate with DUO:

```
(local machine) % ssh YOUR_UMN_ID@stratus-login.msi.umn.edu
```

Then, generate the SSH key. Leave the default paths for the private key.

```
(stratus-login) % ssh-keygen -t rsa -b 4096
```

Then, `cat` the `~/.ssh/id_rsa.pub` file and copy the text from `ssh-rsa` until the end of the line (`. . .umn.edu`).

Switch back to the Openstack Dashboard. Click `Compute` on the left menu, and then choose `Key Pairs`.

![Manage key pairs]({{ "/graphics/stratus_dbgap/add_key_pair.png" | prepend: site.baseurl }})

If you see an old key from yourself there, delete it. Click the `Import Key Pair` button. In the pop-up box that opens, enter a meaningful name. I usually use my UMN ID. In the "Public Key" box, paste the text you copied from the terminal. Click `Import Key Pair`:

![Import key pairs]({{ "/graphics/stratus_dbgap/import_key.png" | prepend: site.baseurl }})

Once your SSH key is imported into the dashboard, you are ready to create and boot a virtual machine.

### <a name="4.3"></a> Part 4.3: Creating a Virtual Machine
To create a new virtual machine, click on `Compute` in the left menu, then choose `Instances`:

![View instances]({{ "/graphics/stratus_dbgap/instances.png" | prepend: site.baseurl }})

The page will update and show you a table of all virtual machines that have been created, along with some information about each one. To make a new virtual machine, click the `Launch Instance` button at the top-right of the page. You should see a pop-up box appear. This is a complicated menu, so we will present it screen-by-screen.

In the first screen (`Details`), enter a name that you like, select `Any Availability Zone` for "Availability Zone," and enter `1` for "Count."

![Details]({{ "/graphics/stratus_dbgap/launch_details.png" | prepend: site.baseurl }})

Then, click `Source` in the left menu. This is where you will set the base disk size and the operating system that your virtual machine will run. Enter these values:

- Select Boot Source: `image`
- Create New Volume: `Yes`
- Volume Size (GB): `20`
- Delete Volume on Instance Delete: `No`.
- Device Name: `vda`
- Image: `Centos7_dbgap_blessed_desktop_ris`; click the little up-arrow on the side to select the image

**Note**: We will make a larger separate volume for the actual data later. This volume is only for the operating system and programs that are installed. Of course, this is not a requirement; it is just my personal preference to separate the operating system files from the dataset files that I will be frequently accessing.

![Source]({{ "/graphics/stratus_dbgap/launch_source.png" | prepend: site.baseurl }})

Then click `Flavor` on the left menu. This is where you will choose the amount of resources that are allocated to your virtual machine. Scroll down to find `m16.small` and click on the little up-arrow on the right side. This will use the entire base allocation of 16 vCPUs and 32GB of memory.

![Flavor]({{ "/graphics/stratus_dbgap/launch_flavor.png" | prepend: site.baseurl }})

Next, click `Security Groups` on the left menu. This is where you will add SSH or remote desktop access to your virtual machine. Find the SSH group you made earlier, and add it to the instance with the up-arrow on the right side of its listing:

![Security Groups]({{ "/graphics/stratus_dbgap/launch_security_groups.png" | prepend: site.baseurl }})

**Note:** If you would like remote desktop access to this virtual machine, you must also add the "RDP" group you made earlier.

Then, click `Key Pair` on the left menu. Make sure your key is in the "allocated" section. Finally, click `Launch` on the bottom-right side of the box!

### <a name="4.4"></a> Part 4.4: Creating a Data Volume
We are not *quite* finished yet! We have to make a volume that will actually hold the data that you will be analyzing. From the Openstack Dashboard, click `Volumes` on the left menu, and then `Volumes` again:

![Volumes]({{ "/graphics/stratus_dbgap/volumes.png" | prepend: site.baseurl }})

Then, click the `Create Volume` button. In the pop-up box that appears, enter the following:

- Volume Name: `vdb`
- Description: `db GaP Data`
- Volume Source: `No source, empty volume`
- Type: `DEFAULT`
- Size (GiB): `1000`

![New volume]({{ "/graphics/stratus_dbgap/new_volume.png" | prepend: site.baseurl }})

Then, in the entry for the new volume, click the little down-arrow on the right side of the table, and choose `Manage Attachments`. In the pop-up box that appears, choose the instance you just launched from the drop-down menu, and enter `/dev/vdb` for "Device Name." Then click `Attach Volume`:

![Attach volume]({{ "/graphics/stratus_dbgap/attach_volume.png" | prepend: site.baseurl }})

Now that your volume is attached, you can actually start to log into the virtual machine!

## <a name="5"></a> Part 5: Accessing a Running Virtual Machine
To access your virtual machine, you will need to record the IP address from the Openstack Dashboard. Click `Compute` in the left menu, and then choose `Instances` to view your list of virtual machines. Take a note of the IP address of the running virtual machine. Its location is shown in the screenshot below:

![Virtual machine IP address]({{ "/graphics/stratus_dbgap/ip_address.png" | prepend: site.baseurl }})

Open a new terminal window and connect to `stratus-login.msi.umn.edu` again:

```
(local machine) % ssh YOUR_UMN_ID@stratus-login.msi.umn.edu
```

You will need to authenticate with DUO. Once you are logged into `stratus-login` you can connect to your virtual machine with the IP address that you recorded from the Openstack Dashboard. I will use a dummy value here, but you should use the real value of your virtual machine. Note that the username of the default user in your virtual machine is `centos` for CentOS images and `ubuntu` for Ubuntu images. It is NOT your UMN Internet ID. Because the RIS image is a CentOS image, we use `centos`:

```
(stratus-login) % ssh centos@10.11.12.123
```

It should automatically connect you to the virtual machine.

<div class="info" markdown="1">

To make it easier to connect to your virtual machine, you can set an alias for yourself on `stratus-login`. Edit your `~/.bashrc` file on `stratus-login` to contain an alias definition. If you want to use a command like `vm_conn` to connect to your virtual machine, you would enter

```
alias 'vm_conn=ssh centos@10.11.12.123'
```

Source your `~/.bashrc` or log out and log in again for the alias to take effect.

</div>

### <a name="5.1"></a> Part 5.1: Mounting the Data Volume
Now that we have connected to the virtual machine, you will have to make a mount point for the block storage volume that you made in section 3.4.

First, verify that the block storage device was successfully attached to the virtual machine. You should see `/dev/vdb` in the listing, which is the name that we assigned the block device:

```
(virtual machine) % ls -l /dev/vd*
brw-rw----. 1 root disk 252,  0 Jan 28 22:46 /dev/vda
brw-rw----. 1 root disk 252,  1 Jan 28 22:46 /dev/vda1
brw-rw----. 1 root disk 252, 16 Jan 28 22:48 /dev/vdb
```

Next, we will make a an EXT4 filesystem on this block device:

```
(virtual machine) % sudo mkfs.ext4 /dev/vdb
```

Then, make the mount point. By convention, we make it under `/mnt`, but you can put it wherever you like. Remember where you put it, though, because this directory is where you will save all of your data files and intermediate analysis files.

```
(virtual machine) % sudo mkdir -p /mnt/dbGaP_Data
```

Next, mount the volume at this mount point:

```
(virtual machine) % sudo mount /dev/vdb /mnt/dbGaP_Data
```

Finally, open the permissions to the newly mounted filesystem so that your regular user account can read and write data:

```
(virtual machine) % sudo chmod -R 777 /mnt/dbGaP_Data
```

## <a name="6"></a> Part 6: Downloading dbGaP Data
If you would like to download dbGaP data as a separate user account (for example, to provide an additional safeguard against accidental deletion or corruption by overwriting), please see [Part 8](#8) of this document!

Your virtual machine should be ready to go when you are, at this point! However, there are some important items to keep in mind when it comes to fetching data from dbGaP. You must use the "AsperaConnect" software to download data from dbGaP, so you must install and configure this software on the virtual machine. Additionally, you will need to make sure that your dbGaP "key file" is available on the virtual machine for decrypting the data once it is downloaded. This section will go over how to configure and use the "AsperaConnect" software for downloading data and how to use your key to decrypt it.

### <a name="6.1"></a> Part 6.1: AsperaConnect
To get the "AsperaConnect" software, open a web browser on your laptop or desktop machine, and go to <https://downloads.asperasoft.com/en/downloads/8?list>. Click "Linux" and then choose the latest version from the drop-down menu. Right-click on the "Direct download" button, and copy the download link.

Switch back to your terminal where you are logged into your virtual machine, and download it with `wget`. Extract the files. We will additionally suggest that you keep all of the software that you download in a special directory on your data volume so it is easy to find later:

```
(virtual machine) % cd /mnt/dbGaP_Data
(virtual machine) % mkdir ./Software
(virtual machine) % cd ./Software
(virtual machine) % wget [paste link here]
(virtual machine) % mkdir ./IBM_AsperaConnect
(virtual machine) % tar -xvzf ibm-aspera-connect-[...].tar.gz -C ./IBM_AsperaConnect
```

Then, run the shell script to install the download client:

```
(virtual machine) % cd ./IBM_AsperaConnect
(virtual machine) % bash ibm-aspera-connect-[...].sh
```

### <a name="6.2"></a> Part 6.2: SRAToolKit
Next, you will additionally need to install SRAToolKit to download data from dbGaP. In a web browser from your desktop or laptop, navigate to <https://trace.ncbi.nlm.nih.gov/Traces/sra/sra.cgi?view=software>. Locate the archive for CentOS Linux 64-bit architecture, and right-click on the link and "copy link."

Switch back to your terminal where you are logged in to your virtual machine. Navigate back to the software directory of your data volume, and download the software with `wget` and extract it:

```
(virtual machine) % cd /mnt/dbGaP_Data/Software
(virtual machine) % wget [paste link here]
(virtual machine) % tar -xvzf sratoolkit.2.10.2-centos_linux64.tar.gz
```

This software should already be compiled and ready to run.

### <a name="6.3"></a> Part 6.3: dbGaP Downloader Key
Next, you must copy your dbGaP downloader key to your virtual machine. This file will let you decrypt the dbGaP data once you download it. To get your key file, go to your "Authorized Requests" page in dbGaP: <https://dbgap.ncbi.nlm.nih.gov/aa/wga.cgi?page=pi_requests>. You need to use your eRA Commons username and password to log in.

Then, clock on the "My Projects" tab to view a list of all of the projects that you are authorized to access. On the right side of the table, click the "get dbGaP repository key" link to download the key file. It should have an extension of `.ngc`

![dbGaP projects]({{ "/graphics/stratus_dbgap/dbgap_my_projects.png" | prepend: site.baseurl }})


Use `scp` to copy the key file to `stratus-login.msi.umn.edu`. Then, you must use `scp` again to copy the key file from the Status bastion host to the virtual machine. For the first copy from your local machine to `stratus-login.msi.umn.edu`, you will need to authenticate with DUO.

```
(local machine) % scp /path/to/key.ngc YOUR_UMN_ID@stratus-login.msi.umn.edu:
(local machine) % ssh YOUR_UMN_ID@stratus-login.msi.umn.edu
(stratus-login) % scp key.ngc centos@10.11.12.123:/mnt/dbGaP_Data/key.ngc
```

where `YOUR_UMN_ID` is your UMN Internet ID.

### <a name="6.4"></a> Part 6.4: Fetching SRA dbGaP Data
Now, you should have all of the requirements in place for actually downloading and decrypting dbGaP data. Go back to the web browser where you are logged into dbGaP. Click the "My Requests" tab, underneath the "Authorized Access" tab.

![dbGaP requests]({{ "/graphics/stratus_dbgap/dbgap_my_requests.png" | prepend: site.baseurl }})

Find the project of your choice in the list of authorized data requests. Click on the "run selector" link:

![dbGaP requests]({{ "/graphics/stratus_dbgap/dbgap_sra_run_selector.png" | prepend: site.baseurl }})

This will take you to the NCBI SRA Run Sector tool, which will let you interactively filter and choose which files you would like to download from the SRA. For example, if your dbGaP project contains both whole genome resequencing (WGS) and RNAseq data, you can use the `Assay Type` filter on the left side of the page to select only one type of data to download:

![dbGaP requests]({{ "/graphics/stratus_dbgap/sra_filter.png" | prepend: site.baseurl }})

If you choose to subset the files (27 RNAseq samples, in this example), be sure to "select" the filtered list by clicking the checkbox in the header row of the sample metadata table displayed at the bottom of the page:

![dbGaP requests]({{ "/graphics/stratus_dbgap/sra_apply_filter.png" | prepend: site.baseurl }})

You can clear the selection by clicking the "X" next to the checkbox. Once you have selected the samples of interest, click the toggle switch in the "Select" box above the metadata table. If the background of the toggle switch is grey, then the selection is not active. If the background of the toggle switch is blue, then the selection is active:

![dbGaP requests]({{ "/graphics/stratus_dbgap/sra_select.png" | prepend: site.baseurl }})

Once the selection is active, click the green "Cart file" button in the center of the "Select" box. This will serve your browser a binary `.krt` file that you can use with the SRA toolkit to download the data. Copy this file to the Stratus login server, then copy it to your virtual machine:

```
(local machine) % scp cart_prjXXXXX_YYYYMMDDHHmm.krt YOUR_UMN_ID@stratus-login.msi.umn.edu:
(local machine) % ssh YOUR_UMN_ID@stratus-login.msi.umn.edu
(stratus-login) % scp cart_prjXXXXX_YYYYMMDDHHmm.krt centos@10.11.12.13:
```

Substitute the appropriate filenames, usernames, and IP addresses for your Stratus instance and user credentials.

<div class="info" markdown="1">

The SRA run selector will give you a cart file that has a name in the following format:

```
cart_prjXXXXX_YYYYMMDDHHmm.krt
```

where `XXXXX` is the dbGaP authorization number, and `YYYYMMDDHHmm` is a time stamp, with `YYYY` representing the year, `MM` representing the month, `DD` representing the day, `HH` representing the hour of the local time, and `mm` representing the minute of the local time.

</div>

Once your `.krt` file has been copied to your virtual machine, `ssh` in to the VM and move it to the same volume as the directory you will use to download your data. For example, if you made a separate data volume and mounted it to `/mnt/dbGaP_Data`:

```
(stratus-login) % ssh centos@10.11.12.13
(virtual machine) % mv cart_prjXXXXX_YYYYMMDDHHmm.krt /mnt/dbGaP_Data/
```

Then, you can use the `prefetch` utility from the SRA toolkit to download the files from the SRA. You will need to supply the dbGaP downloader key as well. Additionally, I have found that it helps to supply a high maximum prefetch file size with the `-X` option, else `prefetch` will skip the file:

```
(virtual machine) % cd /mnt/dbGaP_Data
(virtual machine) % /mnt/dbGaP_Data/Software/sratoolkit.2.xx.0-centos_linux64/bin/prefetch \
    --ngc /path/to/key.ngc \
    -X 100000000 \
    /mnt/dbGaP_Data/cart_prjXXXXX_YYYYMMDDHHmm.krt
```

Depending on how much data is referenced in your `.krt` file, this can take a long time!

### <a name="6.5"></a> Part 6.5: Fetching Non-SRA dbGaP Data
This will look very similar to fetching SRA data, but you will not use the SRA run selector to download kart files. Instead, you will generate a special command to fetch data from NCBI. Access your requests as with SRA data:

![dbGaP requests]({{ "/graphics/stratus_dbgap/dbgap_my_requests.png" | prepend: site.baseurl }})

Find the project of your choice in the list of authorized data requests. Click the link for "Request Files." You will be taken to a page where you can navigate the file structure of the dbGaP project that you are accessing. Check the boxes on the tree to select which files that you would like to download, then click "Create download request."

<div class="warn" markdown="1">

Note that when you create a download request, you will be granted short-term access to the data set. The download request expires after 60 days, so be sure to download your data within 60 days of requesting access. If you do not, then you will have to create a new download request.

</div>

You will then be taken to a page where dbGaP will provide instructions for downloading the data via command line. I have found the "dbgap-aspera-download.pl" script to be more confusing than it's worth, so I use a command based on the one that is listed in the lower box.

Copy the `ascp` command from the box, and paste it into a plain text editor. Replace every instance of `%ASPERA_CONNECT_DIR%` with `/home/centos/.aspera/connect`, and replace all backslashes (`\`) with forward slashes (`/`).

Navigate to the root of the data volume. We will make a new directory here to hold the dbGaP data:

```
(virtual machine) % cd /mnt/dbGaP_Data
(virtual machine) % mkdir ./dbGaP_Workspace
(virtual machine) % cd ./dbGaP_Workspace
```

Copy the modified `ascp` command from your text window and paste it into your terminal in the virtual machine. This will download the data. Depending on the size of your dataset, this may take a while to fetch.

### <a name="6.6"></a> Part 6.6: Decrypting dbGaP Data
Now that you have downloaded some dbGaP data, we will cover how to use your dbGaP key file to decrypt the data. Recall the path to your installation of the SRA toolkit from earlier in this section. Also note the path to the dbGaP workspace that you made above.

NCBI maintains a guide (<https://www.ncbi.nlm.nih.gov/books/NBK63512/>) for decrypting data from dbGaP. Refer to their guide for the latest policies and how to handle edge and corner cases.

You can tell if you have encrypted data that needs to be decrypted if you are working with files that have the `.ncbi_enc` extension.

<div class="warn" markdown="1">

Starting with SRA toolkit version 2.10, the `vdb-config` tool does NOT accept the dbGaP key! Instead, the key file is given to `vdb-decrypt` directly.

If you are following a guide that uses the SRA toolkit 2.9 or earlier, these instructions may not work!

</div>

First, run the `vdb-config` tool with the `-i` option to set the directories for your dbGaP workspace:

```
(virtual machine) % /mnt/dbGaP_Data/Software/sratoolkit.2.10.2-centos_linux64/bin/vdb-config -i
```

This will open an interactive text window where you can set up your workspace parameters. Use the `[TAB]` key to move the cursor around the menus, and use `[SPACE]` or `[ENTER]` to select an item. You can navigate in the reverse direction by holding `[SHIFT]` and pressing `[TAB]`.

Navigate to the `CACHE` tab, and select it. Then, navigate to the `Enable local file-caching` item, and press `[SPACE]` to check its box if it is not yet checked. Enter `/mnt/dbGaP_Data/ncbi` as the cache location. This is where new files will be pre-fetched to, if you need to download additional data or need to fetch `.sra` files through the toolkit:

![vdb-config cache]({{ "/graphics/stratus_dbgap/vdb_config_cache.png" | prepend: site.baseurl }})

Then, navigate to the `AWS` tab. Make sure the box for `accept charges for AWS` is not checked, and that the `report cloud instance identity` is also unchecked. You will not be using AWS.

Then, navigate to the `GCP` tab. This should also be unchecked; you will not be using GCP.

Next, navigate to the `NET` tab. Make sure the `use http-proxy` box is unchecked, and that the `proxy` field is empty. You will not need a proxy to access the internet resources.

Finally, navigate to the `TOOLS` tab. In the `prefetch downloads to` box, check the `user repository` option.

Navigate up to the top menu, and select `[save]`.

Now, you are ready to use the SRA toolkit to decrypt your data and have it set up to pull new data if you need to.

Now, you can decrypt your data with the SRA toolkit. Use the `vdb-decrypt` binary, your downloader key, and the path to the workspace where you fetched the data to decrypt it:

```
(virtual machine) % /mnt/dbGaP_Data/Software/sratoolkit.2.10.2-centos_linux64/bin/vdb-decrypt \
    --ngc /path/to/dbgap_key.ngc \
    /mnt/dbGaP_Data/dbGaP_Workspace
```

Your workspace should now have replaced the `.ncbi_enc` files with standard files, such as `.csv`, `.txt`, or `.tar.gz`, and you can process these with standard Linux utilities or analysis software.

If you are decrpyting data from SRA, you can use the `.ngc` file with `fastq-dump` to extract the FASTQ files:

```
(virtual machine) % /mnt/dbGaP_Data/Software/sratoolkit.2.10.2-centos_linux64/bin/fastq-dump \
    -Q 33 \
    --defline-seq '@$sn[_$rn]/$ri' \
    --defline-qual '+$sn[_$rn]/$ri' \
    --split-files \
    --gzip \
    --ngc /path/to/key.ngc \
    /path/to/archive.sra
```

<div class="info" markdown="1">

The values we are giving to the `--defline-seq` and `--defline-qual` options here are to ensure that the read pairs are properly parsed by downstream read mapping tools. These options will cause the reads to have a format like the following:

```
@1/1
ATCGATCG...
+1/1
xxxxxxxx...
@2/1
ATCGATCG...
+2/1
xxxxxxxx...
```

for read 1, and

```
@1/2
ATCGATCG...
+1/2
xxxxxxxx...
@2/2
ATCGATCG...
+2/2
xxxxxxxx...
```

for read 2. This format is expected by tools such as `bwa` and `hisat2`.

</div>

## <a name="7"></a> Part 7: Installing Software
Now that you know how to download data from dbGaP into your virtual machine, we will cover how to install the software packages that you may need to perform your analyses. For software that offers core functionality, you can use the version that is bundled for CentOS. For specialized bioinformatics analysis software, we provide a set of Conda environment files that will install suites of software based on the type of analysis that you may need to do. These include separate software suites for handling Illumina reads, or genotyping arrays. We will go over how to install software from both the CentOS central repositories, and from our pre-defined Conda environment files.

### <a name="7.1"></a> Part 7.1: Installing With `yum`
CentOS uses the `yum` software management tool. You can use it to install software that provides core functionality to the system. One of the packages I *highly* recommend is GNU screen (<https://www.gnu.org/software/screen/>). To install it with `yum`:

```
(virtual machine) % sudo yum install screen
```

Similarly, you can install command-line text editors:

```
(virtual machine) % sudo yum install emacs
(virtual machine) % sudo yum install vim
```

And GNU Parallel (<https://www.gnu.org/software/parallel/>), a tool for speeding up simple workflows:

```
(virtual machine) % sudo yum install parallel
```

<div class="warn" markdown="1">

The software packages available in the `yum` repositories are not always the most current. If you need the latest version (or just a specific version) of a given piece of software, your best bet is to install it manually. This may require some detailed knowledge of software compilation procedures, though, and MSI cannot support you if you decide to compile your own software from source.

</div>

### <a name="7.2"></a> Part 7.2: RIS Conda Environment Files
The RIS group at MSI maintains a collection of Conda environment files that you can use to easily install suites of software into your virtual machine.

Clone the repository into your virtual machine (may require your UMN InternetID and password):

```
(virtual machine) % git clone https://github.umn.edu/MSI-RIS/RIS_Stratus_Software.git
```

Then, install the `general` environment to get started:

```
(virtual machine) % conda env create -n my_analysis -f ./RIS_Stratus_Software/general.yml
```

This will create a new Conda environment called `my_analysis` based on the packages listed in `general.yml`. You can name the environment whatever you like, but be sure to remember it. You can then "activate" the environment to make the programs available to use via interactive command line or scripts:

```
(virtual machine) % conda activate my_analysis
```

replacing `my_analysis` with the name that you chose when creating it.

You can then "update" the environment with other package lists specified in other environment files. For an end-to-end analysis, you may need several environment files. For example, if you wanted to perform a differential gene expression analysis starting with raw FASTQ data, you would need to install the `ngs-general`, `ngs-aln`, and `ngs-rnaseq` environments:

```
(virtual machine) % conda env update -n my_analysis -f ./RIS_Stratus_Software/ngs-gen.yml
(virtual machine) % conda env update -n my_analysis -f ./RIS_Stratus_Software/ngs-aln.yml
(virtual machine) % conda env update -n my_analysis -f ./RIS_Stratus_Software/ngs-rnaseq.yml
```

<div class="info" markdown="1">

The `ngs-gen` environment contains tools for handling FASTQ data, such as FastQC, Trimmomatic, FASTX-toolkit, and prinseq. The `ngs-aln` environment contains tools for mapping reads to a genome and processing BAM files, such as BWA, Bowtie2, picard-tools, samtools, and sambamba. The `ngs-rnaseq` environment contains tools for processing transcriptomics data, such as Kallisto, RSEM, Subread, and Trinity.

</div>

For a more detailed description of all of the environments available, please see the page at <https://github.umn.edu/MSI-RIS/RIS_Stratus_Software>.

#### A Note about R/Bioconductor
Our Conda environment files do **not** contain any R packages or Bioconductor packages as explicit tools to be installed. However, they may be installed as dependencies (e.g., for GATK or Picard-tools). This is because R already maintains its own environment, and installing Conda-based R packages will create many new R environments, which are likely to be incomplete and incompatible with each other.

Instead, we recommend that you use the **system** R that is installed as part of the RIS Stratus image by calling R by its full path, `/usr/bin/R`. From here, you can install packages like `edgeR` or `DESeq2`, and they will be installed into an environment that is separate from the Conda environments. This will help ensure that your R environment is stable and complete.

Unfortunately, this is one of the downsides of using Conda.

## <a name="8"></a> Part 8: Adding New Users and Groups to Your Virtual Machine
This is a special topic for situations in which multiple users in a PI group need to access a Stratus VM. This may arise, for example, if two researchers under the same dbGaP authorization would like to perform different analyses on the same dataset. Another case is if one person will handle the data downloading and staging, and another person will handle the analyses. In either case, it is helpful to have multiple user accounts for those who are authorized to access the data so each person can manage their software environments and files. It also adds an additional safeguard against accidental data deletion or corruption by allowing you to grant read and write access via UNIX groups, rather than pure trust.

<div class="warn" markdown="1">

By default, only the `centos` user account (or `ubuntu`/`debian` depending on the image you chose to create your VM) can use `sudo` to install system software or perform other superuser actions. You can add other users to the `sudoers` file, but that is outside the scope of this tutorial.

</div>

To add new users and groups to your virtual machine, first connect as the `centos` (or `ubuntu`/`debian`) user account:

```
(local machine) % ssh YOUR_UMN_ID@stratus-login.msi.umn.edu
(stratus-login) % ssh centos@10.11.12.13
```

We will first create the groups that will hold our new users. We will make a group for a dbGaP data downloader (`dbgap_data`) and a group for an analyst (`analysis`):

```
(virtual machine) % sudo groupadd dbgap_data
(virtual machine) % sudo groupadd analysis
```

Then, add a new user with the `useradd` command. In this example, we will add a user account for a downloader, called `dbgap_downloader`, and add this user to the `dbgap_data` group that we made in the previous step. We will also add one for an analyst, called `analyst`:

```
(virtual machine) % sudo useradd -g dbgap_data dbgap_downloader
(virtual machine) % sudo useradd -g analysis analyst
```

Then, set passwords. Note that as you type the passwords, they will not be printed to the terminal. This is normal:

```
(virtual machine) % sudo passwd dbgap_downloader
Changing password for user dbgap_downloader.
New password:
Retype new password:
passwd: all authentication tokens updated successfully.
(virtual machine) % sudo passwd analyst
Changing password for user analyst.
New password:
Retype new password:
passwd: all authentication tokens updated successfully.
```

You can then log out of the VM and test your new accounts. Again, the password will not be displayed as you type it:

```
(virtual machine) % logout
(stratus-login) % ssh analyst@10.11.12.13
analyst@10.11.12.13's password:
Last login: ... from stratus-login.msi.umn.edu
```

The next step is only relevant if you have already downloaded dbGaP data. If you have not already downloaded data, you can perform all the download steps as the `dbgap_downloader` account, and the permissions will automatically be set.

Log out of the virtual machine, and reconnect as the `centos` account. We will change the ownership and permissions on the dbGaP data directory to the `dbgap_downloader` account:

```
(virtual machine) % logout
(stratus-login) % ssh centos@10.11.12.13
(virtual machine) % sudo chown -R dbgap_downloader:dbgap_data /mnt/dbGaP_Data/
(virtual machine) % sudo find /mnt/dbGaP_Data -type f -exec chmod 644 {} \;
(virtual machine) % sudo find /mnt/dbGaP_Data -type d -exec chmod 755 {} \;
```

<div class="info" markdown="1">

The first `sudo find ...` command will find all files (`-type f`) under the `/mnt/dbGaP_Data` directory and change their permission mode to `644`. `644` is read+write for the owner (`dbgap_downloader` in this case), read-only for the group, and read-only for everyone else. This ensures that only the `dbgap_downloader` user account can *change* any of the file contents.

The second `sudo find...` command will find all directories (`-type d`) under the `/mnt/dbGaP_Data` directory and change their permission mode to `755`. `755` is read+write+execute for the owner, read and execute for the group, and read and execute for everyone else. Permission to view directory contents (including `cd` into a directory) is called "execute" in UNIX permission string terms.

This combination of permissions ensures that only the `dbgap_downloader` account has permission to change *any* of the data in the `/mnt/dbGaP_Data` directory, including adding new files or changing the names of files. This helps a lot with making sure the data directory remains "clean" and that timestamps for file modification are preserved.

</div>

## <a name="9"></a> Part 9: Moving Data Out of Stratus
There are two easy ways to move data out of Stratus: via `scp` (for small files) and `s3cmd` (for large files). 

<div class="warn" markdown="1">

Any data that you move out of Stratus must not contain any individual patient-level sequence data or personally-identifiable information. This includes sequence reads (FASTQ), read mapping files (SAM/BAM/CRAM), whole-genome variant call sets (VCF/BCF), personal information (name, address, date of birth, e.g.), and any imagery in which a person can be identified.

</div>

Files that are up to megabytes in size can be easily transferred via `scp` with `stratus-login`:

```
(virtual machine) % scp /path/to/small/file.dat YOUR_UMN_ID@stratus-login.msi.umn.edu:
(local machine) % scp YOUR_UMN_ID@stratus-login.msi.umn.edu:file.dat /path/to/destination
(MSI cluster) % scp YOUR_UMN_ID@stratus-login.msi.umn.edu:file.dat /path/to/destination
```

<div class="warn" markdown="1">

Note that there is a quota of 100MB on `stratus-login`! This host is not meant for large data transfers.

</div>

Files that are larger than several megabytes can be transferred through MSI's Tier2 storage system. To access the Tier2 system from your Stratus VM, you must configure the `s3cmd` command to use MSI's instance. First, get your MSI S3 credentials by logging in to this page: <https://www.msi.umn.edu/content/s3-credentials>. Then, run the `s3cmd` configuration command in the virtual machine. This is an interactive command:

```
(virtual machine) % s3cmd --configure
```

When you see a prompt for `Access Key:`, paste the value from the `Your S3 access key is:` field from the S3 credentials webpage. Paste the value `Your S3 secret key is:` field and the `Secret Key:` prompt. Do not change the default region from `US`.

For the `S3 Endpoint` prompt, enter `s3.msi.umn.edu`. For the `DNS-Style bucket+hostname:port template` prompt, enter `%(bucket)s.s3.msi.umn.edu`.

For the `Encryption password` prompt, you can enter any password you like. **NOTE** though that this password will show up in plain text. Do not change the path to the GPG program from its default.

Leave the defaults for the other prompts, then let `s3cmd` test the connection. If you get an error message or a message that the connection failed, check the spellings of your S3 access key, S3 secret key, and the endpoint name. If those are correct, but you still get an error message, please contact the MSI help desk!

Then, use `s3cmd mb` and `s3cmd put` to make buckets in your Tier2 storage space and deposit data:

```
(virtual machine) % s3cmd mb s3://stratus-xfer
(virtual machine) % s3cmd put -v /path/to/big/file.dat s3://stratus-xfer/
```

You can then pull the data to your MSI home directory:

```
(MSI cluster) % s3cmd get -v s3://stratus-xfer/file.dat /path/to/destination
```

## <a name="10"></a> Part 10: Advanced Tips and Tricks
Note that none of the content in this section is *required* for working with Stratus! These are more advanced tricks for working with Linux systems. On the primary MSI systems, these are handled by a team of professional system administrators, but when using Stratus, you may have to deal with these on your own.

### <a name="10.1"></a> Part 10.1: Making a Swapfile
The recommended VM image does not come with a swap partition, so you may want to make a swap space for your work.

<div class="info" markdown="1">

A swap partition is a space on disk that can be used like an extension of RAM. When RAM gets full, less-active parts of memory are written to the disk ("swapped out") to make room for more active memory.

</div>

If you are encountering out-of-memory errors with your work, you can try to add swap space before purchasing expansions to memory on your Stratus subscription. To make a swapfile, run the following:

```
(virtual machine) % sudo dd if=/dev/zero of=/swapfile bs=1M count=8192
(virtual machine) % sudo chown root:root /swapfile
(virtual machine) % sudo chmod 600 /swapfile
(virtual machine) % sudo mkswap /swapfile
(virtual machine) % sudo swapon /swapfile
```

These commands will make a file that is 8GB in size, turn it into a swapfile, then activate it. You can verify that it is working by looking at the `/proc/swaps` file:

```
(virtual machine) % cat /proc/swaps
Filename                Type        Size    Used    Priority
/swapfile               file        8388604 4240632 -2
```

This information will also be displayed in the output of `top`:

```
(virtual machine) % top -b -n 1
top - 16:18:38 up 29 days, 18:20,  1 user,  load average: 5.99, 6.04, 5.91
Tasks: 258 total,   3 running, 253 sleeping,   0 stopped,   2 zombie
%Cpu(s): 37.6 us,  1.3 sy,  0.0 ni, 60.8 id,  0.3 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem : 32778384 total,   234028 free, 32091340 used,   453016 buff/cache
KiB Swap:  8388604 total,  4128968 free,  4259636 used.   322448 avail Mem

...
```

<div class="warn" markdown="1">

Swap space is NOT the solution to needing additional RAM! It can help in the cases where you know your analyses will use almost the total amount of RAM in your VM. If you try to use swap space in place of RAM, your VM will start to "thrash," which means it is constantly "swapping in" and "swapping out." This will make the VM unstable and will also make your analyses take much longer to finish.

</div>

### <a name="10.2"></a> Part 10.2: Managing Disk Input/Output
With Stratus, you have to be a bit more careful with managing disk input/output (I/O) than with MSI's primary systems. You still have to be careful wth the primary systems, but they seem to have *much* higher bandwidth than the Stratus VMs. If you notice that your processes are running much more slowly on Stratus than you would expect, then you should check the disk activity. If the disk read and/or write bandwidth is fully used, then other processes that need to read data from disk will have to wait, slowing your analyses.

To see if you have processes that are stalled and waiting for data from disk, you can look at the `S` column in the output of `top`. If a process has `D` (uninterruptible sleep), then that is a sign that it is stalled for disk I/O reasons:

```
(virtual machine) % top -u centos -n 1 -b
top - 17:48:38 up 37 days, 19:50,  1 user,  load average: 1.09, 1.12, 1.13
Tasks: 253 total,   3 running, 250 sleeping,   0 stopped,   0 zombie
%Cpu(s):  5.7 us,  0.4 sy,  0.0 ni, 93.4 id,  0.4 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem : 32778384 total,   219208 free,  1705820 used, 30853356 buff/cache
KiB Swap:  8388604 total,  8018460 free,   370144 used. 30621740 avail Mem

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
 1554 centos    20   0 1593488   1.1g   2520 D  81.4  3.5  14:55.51 some-program...
...
```

<div class="info" markdown="1">

There are multiple reasons why a process may be in uninterruptible sleep, but disk I/O limitations is the one we are considering here. Technically, any process that needs to make an uninterruptible system call enters the `D` state, but in a healthy system, it lasts microseconds and will not slow down your work.

</div>

You can use the `iotop` tool to check which processes are using a lot of disk bandwidth. This tool shows output similar to what `top` shows, but for disk activity. `iotop` is not installed by default, so we will use `yum` to install it. It also requires root privileges:

```
(virtual machine) % sudo yum install iotop
(virtual machine) % sudo iotop -u centos
Total DISK READ :      31.95 M/s | Total DISK WRITE :    1380.73 K/s
Actual DISK READ:      31.86 M/s | Actual DISK WRITE:       0.00 B/s
  TID  PRIO  USER     DISK READ  DISK WRITE  SWAPIN     IO>    COMMAND
 1554 be/4 centos     31.95 M/s    0.00 B/s  0.00 %  8.20 % some-program...
 1553 be/4 centos      0.00 B/s 1380.73 K/s  0.00 %  0.00 % another-program...
```

Here, we see that two processes, `some-program` and `another-program` are using a lot of the disk I/O. If these are not critical processes to either the system or the analysis, you can kill them. If they are critical to the analysis, then you will have to schedule them such that they do not exceed the resources of the VM. If they are critical to the system, then you might consider rebooting the VM.

To reduce the number of processes that are trying to read from disk at once, you can try to reduce the number of concurrent processing steps that you are running. For example, if you are trying to convert multiple SAM files to BAM files at once, you should try to reduce the number of file processing commands you are running in parallel. If you are calling variants with a multi-threaded program, you can try to reduce the number of threads the program uses.

## <a name="11"></a> Part 11: Feedback
This tutorial document was prepared by Thomas Kono, in the RIS group at MSI.
Please send feedback and comments to konox006 [at] umn.edu. You may also send
tutorial delivery feedback to that address.

[Return to top](#top)
