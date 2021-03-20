---
layout: collection
collection: tutorials
title: Workspace Setup(ongoing)
category: Guide for Microbiome Data Analysis
date: 2019-09-05
published: false
excerpt: Learn basic concepts and HOWTOs, which are required to manually setup your own workspace for processing & analysis of amplicon metagenomics data.
tags:
    - tutorial
    - microbiome
    - amplicon-metagenomics
    - data-analysis
    - workspace
---

Background
==========

Because of the nature of a bioinformatics field, you are going to need
to understand several concepts in the field of “Computer Science".
Particularly, you will need to organize your *workspaces* in the best
way you can so that you can be more efficient in your work. The word
*environment* which is also known as workspace, can have different
meanings. However, in order to get rid of confusion assume that
environment means **virtual-environment**, but then again, what is a
virtual-environment?

Environments and Package Managers
---------------------------------

In order to understand virtual-environments, lets make a simple analogy.
Think about how you organize your own files in Microsoft Windows or any
other operating system(OS). For example, you naturally make effort to
keep the documents related to your master thesis or PhD research in
separate folders. In such case you are actually creating a kind of
virtual environments or workspace to prevent confusion that you may face
later on. Similarly, so called virtual-environments help you to keep
your software or packages(ie. R or Python packages) in different
environments so that any compatibility issues can be avoided. But why
actually it is necessary to keep them separately? To answer this
question you first need to understand **package-managers**.

### Package Managers

Typically, when you want to install some software into your computer
with Windows OS, there are some common steps that you follow. For
example, first you obtain it from some source such as a USB flash drive
that you got from your friend or an online web-page where you have
downloaded it. Here, by *it*, I mean the installation file(or files)
that represent your software and usually has file extension of *.exe*
aka executable. Next, you open(or execute) the .exe file and so called
installation wizard window pops up that interactively guides your
installation process. This installation wizards is actually functionally
very similar to a package manager but not they are not the same. What I
mean here by saying "not the same"? Well, explaining it would go beyond
the scope of this guide. But in order to give you an idea, just know
that any software, package or even some piece of programming code,
always depends on something else within your system. Similarly, rarely
any software can function completely independent of some
**dependencies**, which are actually other software, libraries,
programs, etc. Rather most of the time when you install some software
like “Microsoft Office" you automatically install many other different
programs that are necessary in order to make your desired software to
work properly. Usually all of these dependencies issues are handled by
so called package-managers.

Package-managers actually handle not only all of the software dependency
issues, but also check your software-hardware or software-software
compatibility, let you update or downgrade your software to new or older
version, delete it safely from your system, report you if any
installation problem occurs, keep all the error logs, etc. Basically
without them world would be in a terrible chaos and even though they
will eventually make you cry a lot, you should love package-managers
without conditions. Also package-managers are universal and all
operating systems have at some kind of package-manager. However, usually
package mangers do not have a user interface with buttons, text-boxes,
etc. like in Windows OS and most of the time only available interface is
a **command-line interface**(aka **terminal** in linux). However, this
fact should not stress you because at the first glance command-line
interface seems disgusting, but later it become irreplaceable. I will
not discuss details of command-lines interfaces here because they
require a separate tutorial. However, I would like to point out that
despite the fact that both Windows and Linux OS have command-line
interface, functionally latter one is way more powerful than the first
one.

### Virtual Environment

Now that you have better idea about package-manager we can answer to
original question about what is a virtual-environment. Well say that you
need to install some latest version tools for your research but you also
need to install same tools but older versions. Now you have a problem
because basic package manager will not let you install both at the same
time since different versions of same packages cause compatibility
problems. As a solution you can use power of virtual-environments
because each virtual-environment is actually a separate workspace that
never interact with other virtual-environments. This is very useful
feature and let’s you to use different tools/software/packages without
damaging your operating system and without need for second computer.

There are great number of different package-managers that have very
different features and applications but basically they have similar
objectives. Some tools are not only package-managers but also
virtual-environment managers. Some tools can even provide much more
features. Here are few examples:

-   Internal package managers used in different operating systems.

    -   **apt-get** and **dpkg** - Default package managers for Debian
        based Linux operating systems such as Ubuntu.
    -   **rpm** - Default package manager for Fedora based Linux
        operating systems such as OpenSuSE.

-   Package managers that have nothing to do with operating systems

    -   **pip** - Recommended Python package manager.
    -   install.packages - Default R package manager.
    -   **conda** - Extremely useful package and virtual-environment
        manager originally intended for Python only but now can also
        manage packages for R, Java, C/C++, etc.
    -   **Docker** - Nowadays is a very popular and useful set of tools
        that provide platform as a service (PaaS). Docker is more than
        just a package or virtual-environment manager since it can
        manage virtual operating systems much more efficiently than
        typical virtual operating systems managers such as VirtualBox.

Now that you have better idea about package-managers and
virtual-environments lets understand what is a **repository**(aka
**repo**).

### Repository

For users of Microsoft Windows operating system concept of repository
seems distant. However, easiest(but not very accurate) way to understand
idea of repository, is to pick up your smart-phone and try to install
any application. You naturally will open Google Play to download and
install applications for Android. Similarly, if you have iPhone you will
open Apple AppStore to find, download and install your application. Both
Google Play and AppStore are actually kind of repositories. Both check
if your smartphone device support the applications, download tool and
then install it. The main difference is that both Google Play and
AppStore are universal and any application that is not registered in
either Google Play and AppStore are typically considered as potentially
dangerous and not recommended by the vendors. This software installation
approach is very different to users of Windows OS. Typically, to install
some software to Windows OS you need to manually find and download or
buy software, and then install it. This is actually main reason why
Windows OS has so many malware like viruses, Trojans or similar. When
you use repository it is up to you whether you trust it or not. However,
most popular or official repositories can be trusted because they are
constantly verified by community members.

### Summary

To sum up, package-managers are extremely useful tools that can manager
your software installations for you. Some package-managers can also
manage virtual-environments for you, although latter not necessarily
always go in parallel with former. In contrast to virtual-environments,
all true package-managers almost always work closely with software
repositories. Finally, terminology for the package-managers,
virtual-environments and repositories can differ. For example, in the
scope of the *conda* package and virtual-environment manager, a
repository is also known as a *channel*.

The power of Conda
------------------

From this point and on, I will only focus on using conda package-manager
to configure and setup up your workspace.

### Why Conda?

Workspace Setup
===============

Install Conda Package Manager
-----------------------------

Miniconda3 (for Python3.7): Free minimal installer for conda.

1.  Go to your “Download" directory: `cd ~/Downloads`

2.  Download miniconda:
    `wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh`

3.  Add executable permission to the downloaded file:
    `chmod +x Miniconda3-latest-Linux-x86\_64.sh`

4.  Execute the installation file as root:
    `sudo ./Miniconda3-latest-Linux-x86\_64.sh`

5.  Quit the terminal: `exit`

6.  Open terminal again

Setup Virtual Conda Environment for QIIME2
------------------------------------------

QIIME2: Quantitative Insights Into Microbial Ecology

1.  Go to do downloads directory: `cd ~/Downloads`

2.  Download QIIME2 conda environment:
    `wget https://data.qiime2.org/distro/core/qiime2-2019.7-py36-linux-conda.yml`

3.  Create and initiate new conda environment called qiime2:
    `conda env create -n qiime2 --file qiime2-2019.7-py36-linux-conda.yml`

4.  Activate new conda environment: `conda activate qiime2`

5.  Return to base environment: `conda activate base`

Setup Virtual Conda Environment for Other Tools
-----------------------------------------------

1.  Create new conda environment with called “genom"

    1.  Create new “genom" environment: `conda create --name genom`

    2.  Activate “genom” environment: `conda activate genom`

2.  Trimmomatic: A flexible read trimming tool for Illumina NGS data

    1.  Make sure that you are in “genom” environment. If not
        then:`conda activate genom`

    2.  Install Trimmomatic from bioconda channel:
        `conda install -c bioconda trimmomatic`

3.  FastQC: A quality control application for high throughput sequence
    data

    1.  Make sure that you are in “genom” environment. If not
        then:`conda activate genom`

    2.  Install FASTQC from bioconda channel:
        `conda install -c bioconda fastqc`

4.  EMBOSS: The European Molecular Biology Open Software Suite

    1.  Make sure that you are in “genom” environment. If not
        then:`conda activate genom`

    2.  Install EMBOSS from bioconda
        channel:` conda install -c bioconda emboss`

5.  FASTX-Toolkit: Collection of command line tools for Short-Reads
    FASTA/FASTQ files preprocessing.

    1.  Make sure that you are in “genom” environment. If not
        then:`conda activate genom`

    2.  Install FASTX from bioconda channel:
        `conda install -c bioconda fastx\_toolkit`

6.  VSEARCH: A versatile open source tool for metagenomics (USEARCH
    alternative)

    1.  Make sure that you are in “genom” environment. If not
        then:`conda activate genom`

    2.  Install VSEARCH from bioconda channel:
        `conda install -c bioconda vsearch`

7.  SeqKit: A cross-platform and ultrafast toolkit for FASTA/Q file
    manipulation in Golang

    1.  Make sure that you are in “genom” environment. If not
        then:`conda activate genom`

    2.  Install SeqKit from bioconda channel:
        `conda install -c bioconda seqkit`

