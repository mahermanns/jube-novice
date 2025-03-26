---
title: "Working with JUBE"
teaching: 10
exercises: 10
---

:::::::::::::::::::::::::::::::::::::: questions

- Which actions need to be taken when running JUBE?
- Which commands correspond to which state in a JUBE workflow?

::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: objectives

- List the basic commands for running a JUBE workflow.
- Specify the purpose of each top-level command.
- Name the different states a JUBE workflow can have.

::::::::::::::::::::::::::::::::::::::::::::::::

JUBE workflows are started with the command `jube run <jube-spec>`.
The configuration file can be either an XML or a YAML file.

A minimal configuration in XML contains the tags `<jube>` and `<benchmark>`.

::: group-tab

### XML

```xml
<?xml version="1.0" encoding="UTF-8"?>
<jube>
  <benchmark name="hello_world" outpath="bench_run">
    <comment>A simple workflow example</comment>
    <!-- further configuration goes here -->
  </benchmark>
</jube>
```

### YAML

```yaml
name: hello_world
outpath: bench_run
comment: A simple workflow example
```
:::

This minimal configuration does not include any definition of workflow steps and the output will look similar to the following:

```output
######################################################################
# benchmark: hello_world
# id: 0
#
# A simple workflow example
######################################################################

Running workpackages (#=done, 0=wait, E=error):
............................................................ (  0/  0)

  | stepname | all | open | wait | error | done |
  |----------|-----|------|------|-------|------|

>>>> Benchmark information and further useful commands:
>>>>       id: 0
>>>>   handle: episodes/files/bench_run
>>>>      dir: episodes/files/bench_run/000000
>>>>  analyse: jube analyse episodes/files/bench_run --id 0
>>>>   result: jube result episodes/files/bench_run --id 0
>>>>     info: jube info episodes/files/bench_run --id 0
>>>>      log: jube log episodes/files/bench_run --id 0
######################################################################
```

## Defining a workflow

With no *steps* defined, the workflow will immediatley complete without specific actions executed, other than file JUBE-internal handling of the run and creation of corresponding directories and files.

Looking into the benchmark output directory created, we will see somethin that will look similar to the following stucture:

```sh
$ tree bench_run        # the tree command provides a nice tree for the directory structure
```
```output
bench_run               # the given outpath
|
+- 000000               # the benchmark id
   |
   +- analyse.log       # accumulated log information for analyse command
   +- analyse.xml       # the retrieved analyse information
   +- configuration.xml # the stored benchmark configuration
   +- parse.log         # log information
   +- result.log        # accumulated log information for result command
   +- run.log           # accumulated log information for run command
   +- timestamps        # accumulated of change timestamps
   +- workpackages.xml  # workpackage information

```

### Log Files
The files ending in `.log` collect details on the execution of the corresponding commands.
Users will only be interested in evaluating these files when debugging a workflow.

### Configuration Files
The files ending in `.xml` save configuration information of the workflow.
The file `configuration.xml` holds a copy of the overall configuration used to start the workflow.
The file `workpackages.xml` holds information about individual instances of parameters in a specific step of the workflow.

Before running a workflow, at least a single *step* needs to be defined for the workflow.
A step is a collection of tasks (defined by individual *do* directives) that share a specific instance of parameter values.
By allowing parameters to have different values in different instances, JUBE will automatically save a snapshot of the collection of all parameter values across all instances.


## Defining parameters

Next to defining the steps of a workflow, *parameters* can both represent values picked up during the workflow and influence how a workflow is executed.
To make it easier to manage multiple *parameters*, you need to organize them in *parameter sets*.
Let's define a minimal set of parameters for our initial *Hello World* example.

::: group-tab

### XML

```xml
<?xml version="1.0" encoding="UTF-8"?>
<jube>
  <benchmark name="hello_world" outpath="bench_run">
    <comment>A simple workflow example</comment>
    <!-- further configuration goes here -->
  </benchmark>
  <parameterset>
    <parameter name="message">Hello World</parameter>
  </parameterset>
</jube>
```

### YAML

```yaml
name: hello_world
outpath: bench_run
comment: A simple workflow example
parameterset:
  name: hello_pset
  parameter: {name: message, _: Hello World}
```
:::

The only reason to define parameters in the first place is to *use* them in the execution of workflow steps.
To do this, the keyword `use`, referencing the name of a parameter set, is needed in the step definintion.


::: group-tab

### XML

```xml
<?xml version="1.0" encoding="UTF-8"?>
<jube>
  <benchmark name="hello_world" outpath="bench_run">
    <comment>A simple workflow example</comment>
    <!-- further configuration goes here -->
  </benchmark>
  <parameterset name="hello_pset">
    <parameter name="message">Hello World</parameter>
  </parameterset>
  <step name="print_message">
    <use>hello_pset</use>
    <do>echo $message</do>
  </step>
</jube>
```

### YAML

```yaml
name: hello_world
outpath: bench_run
comment: A simple workflow example
parameterset:
  name: hello_pset
  parameter: {name: message, _: Hello World}
step:
  name: print_message
  use: hello_pset
  do: echo $message
```
:::

## Running a workflow

A JUBE workflow is started by the `run` command, putting this workflow into the
`running` state.
For each defined step, individual workpackages are created with a specific instance of parameter values.
Any defined tasks
When no *asynchronous* tasks are defined (See [Running an
application](../episodes/run_step.md)) the steps will be processes until all workpackages are completed.

![The JUBE workflow with `run`,
`continue`, `analysis`, and `result` commands.](fig/JUBE_Workflow.svg){alt='The JUBE workflow with `run`,
`continue`, `analysis`, and `result` commands.' width='100%'}



::::::::::::::::::::::::::::::::::::: keypoints

- The different states of a JUBE workflow can be `running` and `completed`.
- A workflow is completed if all steps of the workflow are completed (even if some steps may have failed).
- A JUBE workflow is started with `jube run`.
- JUBE executes the workflow until the end, or until an asynchronous task is discovered.
- If JUBE exits with completion of asynchronous tasks still pending, the user needs to call `jube continue` to check for their completion and further running of dependent workflow steps until overall completion is achieved.
- The user can analyze output and print results at any time.

::::::::::::::::::::::::::::::::::::::::::::::::

