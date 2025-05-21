---
title: "Thoughts on reproducibility"
teaching: 5
exercises: 15
---


:::::::::::::::::::::::::::::::::::::: questions

- How can I improve reproducibility of my JUBE workflow?

::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: objectives

- Identify key aspects of the workflow and its parameters.
- Define ways to record values of these aspects as part of a run.

::::::::::::::::::::::::::::::::::::::::::::::::

JUBE in itself does **not** make every workflow definition inherently reproducible.
However, all parts of a workflow definition and parameter evaluation are recorded and retained during workflow execution.
To increase reproducibility of a workflow, it's the user's repsonsibility to identify all aspects of a workflow and define ways to record them during execution of the workflow.

::::::::::::::::::::::::::::: discussion

Which aspects of the build and measurement steps should be recorded to increase reproducibility of the workflow?

:::::::::::::::::::::::::::::::::::::::
::::::::::::::::::: solution

Potential aspects that merit recording:

- For general workflow execution
 - Build flags used to generate binaries
 - Versions of software and libraries used
 - Input files and configuration
 - Output files
 - Execution scale and domain decomposition
 - ...
- For performance measurements (additionally to the ones above)
  - Networking libraries involved
  - Configuration of networking library
  - Processor model and microcode version
  - OS and kernel version
  - Node names used for the execution
  - Execution configuration (pinning, distribution, ...)
  - ...

::::::::::::::::::::::::::::

:::::::::::::::::::::: challenge
Select one or more of the aspects identified in the prior discussion and specify a configuration to record it as part of a workflow.
::::::::::::::::::::::::::::::::
:::::::::::::::::::::: solution
:::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: keypoints

- **Identify aspects** of the workflow that are important for reproducibility.
- Obtain values for these either through **parameters** or **patterns**.
- **Add actions** to the workflow that help in recording information through
  patterns.

::::::::::::::::::::::::::::::::::::::::::::::::




