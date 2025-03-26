---
title: Setup
---

This lessons focuses on execution of workflows on HPC systems, including
interaction with its job scheduler. You will therefore have to

1. have access to a system with a job scheduler (e.g., SLURM), and
2. setup the JUBE software environment on that system.

## Data Sets

This lesson uses the [GROMACS software][gromacs-home] as an example HPC code.
GROMACS is a free and open-source software suite for high-performance molecular dynamics and output analysis.

Eventually, downloading GROMACS will be part of the overall HPC workflow, but
you can download the 2024.5 version of GROMACS from the [download page of GROMACS][gromacs-download].



## Software Setup

The `environment.yaml` file describes a Conda virtual environment that includes R, pandoc, and termplotlib: the tools you'll need to develop and run this lesson, as well as some depencencies. To prepare the environment, install Miniconda following the official instructions. Then open a shell application and create a new environment:

```sh
user@cluster:~$ cd path/to/local/jube-novice
user@cluster:jube-novice$ conda env create -f environment.yaml
```

N.B.: the environment will be named "jube-novice" by default. If you prefer another name, add `-n <alternate_name>` to the command.

