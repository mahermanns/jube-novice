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

Data sets for this lesson are [packaged with this lesson and can be downloaded
in preparation][data-tarball].


## Software Setup

:::::::::::: tab

### Conda

The `environment.yaml` file describes a Conda virtual environment that includes JUBE: the tool you'll need to run this lesson, as well as some depencencies.
To prepare the environment, [install Miniconda following the official instructions][miniconda-install].
Then download the file [environment.yaml](files/environment.yaml), which contains
all necessary Python packages for this course.
Finally, open a shell application and create a new environment:

```sh
user@cluster:~$ cd path/to/local/jube-novice
user@cluster:jube-novice$ conda env create -f environment.yaml
```

**Note:** The environment will be named "jube-novice" by default. If you prefer another name, add `-n <alternate_name>` to the command.

### Python Virtual Env

The `requirements.txt` file describes a Python virtual environment that
references JUBE: the tool you'll need to run this lesson, as well as some
dependencies.
To prepare the environment, create a Python virtual environment first, activate
and update pip, before installing jube.

```sh
user@cluster:~$ python -m venv jube-venv
user@cluster:~$ source jube-venv/bin/activate
user@cluster:~$ pip install --upgrade pip
user@cluster:~$ pip install http://apps.fz-juelich.de/jsc/jube/download.php?version=latest
```

:::::::::::::::::
