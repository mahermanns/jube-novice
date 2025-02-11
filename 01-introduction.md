---
title: "Introduction"
teaching: 10
exercises: 2
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

It is important to understand that JUBE, while emerged as a benchmarking system, actually is a workflow management system, where application and system benchmarking isone of its applications.
Further use cases may include running multiple HPC workflows as integration tests during application development and defining reproducible execution workflows for data generation for publications.

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



::::::::::::::::::::::::::::::::::::: keypoints

- Workflow automation can assist in getting reproducible results.
- JUBE lets you execute and manage complex workflows on HPC systems.
- JUBE eases exploration of a large parameter space of measurements.
- JUBE automatically isolates directories of distinct steps of a run to avoid
  steps running in parallel to overwrite or unintentional reuse of intermediate
  data.


::::::::::::::::::::::::::::::::::::::::::::::::



