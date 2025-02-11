---
title: "Working with JUBE"
teaching: 10
exercises: 2
---

:::::::::::::::::::::::::::::::::::::: questions 

- Which steps need to be taken when running JUBE?

::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: objectives

- Explain how to use markdown with The Carpentries Workbench
- Demonstrate how to include pieces of code, figures, and nested challenge blocks

::::::::::::::::::::::::::::::::::::::::::::::::

JUBE workflows are started with the command `jube run <jube-spec>`.
The configuration file can be either an XML or a YAML file.

A minimal configuration in XML contains the tags `<jube>` and `<benchmark>`.

:::::::::::::::::::::::::::::: spoiler

### XML

```xml
<?xml version="1.0" encoding="UTF-8"?>
<jube>
  <benchmark name="hello_world" outpath="bench_run">
    <!-- further configuration goes here -->
  </benchmark>
</jube>
```

::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::: spoiler

### YAML

```yaml
name: hello_world
outpath: bench_run
```

::::::::::::::::::::::::::::::::::::::::

## Running a workflow

A JUBE workflow is started by the `run` command, putting this workflow into the
`running` state.
Any defined tasks
When no asynchronous tasks are defined (See [Running an
application](../episodes/07-benchmark-run.md)) the steps wi.

![The JUBE workflow with `run`,
`continue`, `analysis`, and `result` commands.](fig/JUBE_Workflow.svg){alt='The JUBE workflow with `run`,
`continue`, `analysis`, and `result` commands.' width='100%'}




::::::::::::::::::::::::::::::::::::: keypoints

- A JUBE workflow is started with `jube run`
- A JUBE workflow is completed when all defined steps completed.

::::::::::::::::::::::::::::::::::::::::::::::::

