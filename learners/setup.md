---
title: Setup
---

This lessons focuses on execution of workflows on HPC systems, including
interaction with its job scheduler. You will therefore

1. have access to a system with a job scheduler (e.g., SLURM), and
2. setup the JUBE software environment on that system.

## Data Sets

This lesson uses the [GROMACS software](https://www.gromacs.org/) as an example 
GROMACS is a free and open-source software suite for high-performance molecular dynamics and output analysis.

Eventually, downloading GROMACS will be part of the overall HPC workflow, but
you can download the 2024.5 version of GROMACS from the [download page of GROMACS](https://manual.gromacs.org/2024.5/download.html).



## Software Setup

The `environment.yaml` file describes a Conda virtual environment that includes R, pandoc, and termplotlib: the tools you'll need to develop and run this lesson, as well as some depencencies. To prepare the environment, install Miniconda following the official instructions. Then open a shell application and create a new environment:

```sh
user@cluster:~$ cd path/to/local/jube-novice
user@cluster:jube-novice$ conda env create -f environment.yaml
```

N.B.: the environment will be named "jube-novice" by default. If you prefer another name, add `-n <alternate_name>` to the command.

::::::::::::::::::::::::::::::::::::::: discussion

### Details

Setup for different systems can be presented in dropdown menus via a `spoiler`
tag. They will join to this discussion block, so you can give a general overview
of the software used in this lesson here and fill out the individual operating
systems (and potentially add more, e.g. online setup) in the solutions blocks.

:::::::::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::: spoiler

### Windows

Use PuTTY

::::::::::::::::::::::::

:::::::::::::::: spoiler

### MacOS

Use Terminal.app

::::::::::::::::::::::::


:::::::::::::::: spoiler

### Linux

Use Terminal

::::::::::::::::::::::::

