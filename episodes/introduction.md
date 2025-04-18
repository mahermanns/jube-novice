---
title: "Introduction"
teaching: 10
exercises: 10
---

:::::::::::::::::::::::::::::::::::::: questions 

- How do I manage to run an HPC workflow reproducibly?
- How do I easily create a scaling experiment for an HPC application?
- How do I easily identify the optimal runtime configuration for an application
  on an HPC system?

::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: objectives

- List benefits of workflow automation.
- Describe potential use cases for HPC workflow automation using JUBE.

::::::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::: instructor

The key goal of this episode is to provide potential uses cases for JUBE, and
enable the participants to transfer these to their own workflows on HPC
systems.

::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::

When applying for computing time on HPC systems, applicants are often asked to provide measurements on different scales of parallelism.
Furthermore, preparing performance measurements often involves an application-specific workflow as well as platform-specific configurations.
The objective of this lesson is to enable users of HPC systems to run performance measurements with minimal intervention with high reproducibility, using the [Jülich Benchemarking Environment (JUBE)](https://apps.fz-juelich.de/jsc/jube/docu/index.html) [1].

It is important to understand that JUBE, while emerged as a benchmarking system, actually is a workflow management system, where application and system benchmarking is one of its applications.
Further use cases may include running multiple HPC workflows as integration tests during application development and defining reproducible execution workflows for data generation for publications.

::: instructor
It is important for the learners to understand that JUBE itself **does know what
make your workflow reproducible on its own**. 
To create a reproducible workflow learners will have to **identify which
information and actions make the specific workflow reproducible**, and use JUBE
to **automatically collect and execute** those, respectively.
:::

::: caution

JUBE is an automation tool. That means, it **does not** intrinsicly make
your workflows reproducible, but it **lets you automate** all actions that
can make your workflow reproducible.

:::

The key benefits of using JUBE are:

- It enables systematic and reproducible benchmarks.
- It provides a flexible, script-based framework to set up tasks and control
  various aspects of the execution.
- It allows for custom workflows that can adapt to different HPC platforms.
- It provides a powerful tagging systems to influence execution of a given
  workflow.
- It automatically isolates your workflow steps so concurrent tasks don't
  overwrite or re-use files in unintentionally shared execution environments.
- It enables efficient parameter space exploration.
- It allows to easily retrieve strings from runtime output and create tables
  and CSV files.

::::::::::::::::::::::::::::::::::::: discussion

Think of your own workflows on an HPC system.
What individual steps are involved?
Discuss in a group.

:::::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::: solution

Steps may include:

- Loading and preparing the software environment
- Building a software package
- Creating a runtime configuration for your application
- Creating a scheduler job script
- Submitting a job to the scheduler
- Extracting strings from application output.

:::::::::::::::::::::::::::::::::

[JUBE](https://www.fz-juelich.de/jsc/jube/) is written and maintained by the [Jülich Supercomputing
Centre](https://www.fz-juelich.de/jsc/) at [Forschungszentrum
Jülich](https://www.fz-juelich.de/) in Germany.
It is written in Python 3, but is backwards-compatible to Python 2.x.
The user can write configurations in XML or YAML syntax.
It is freely [available for download](https://www.fz-juelich.de/en/ias/jsc/services/user-support/software-tools/jube/download) and also available via HPC software package managers, such as [Spack](https://spack.io/) and [EasyBuild](https://easybuild.io/).


## Preparing your workspace

In preparation of following exercises, we will create a specific workspace for our lesson workflow.

::: challenge

### Creating a workspace for the lesson's exercises

It is best to start with an empty directory. You are free to choose any name,
but for the sake of this lesson, we will use `jube-workspace/`.

```sh
$ mkdir jube-workspace/
$ cd jube-workspace/
```

```output
jube-workspace$
```

:::


::::::::::::::::::::::::::::::::::::: keypoints

- Workflow automation can assist in getting reproducible results.
- JUBE lets you execute and manage complex workflows on HPC systems.
- JUBE eases exploration of a large parameter space of measurements.
- JUBE automatically isolates directories of distinct steps of a run to avoid
  steps running in parallel to overwrite or unintentional reuse of intermediate
  data.


::::::::::::::::::::::::::::::::::::::::::::::::



