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

## Query the status of a workflow

To check whether a run with a given id if finished or not, you can use the `jube status` command. Oviously, the workflow you just executed is finished.

```sh
$ jube status --id 0
```
```output
FINISHED
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

Instead of opening the file directly (which would be possible), you can also use the `jube log` command to show the log of a given command.

```sh
$ jube log --command run --id <id>
```
```output
tbd.
```

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

:::::: callout

## Naming conventions

Several entities defined in the specification are later **used** by its name in the step definitions.
To make it easier to understand more complex configurations, it is good practice to encode the type of the entity into its name, for example as a suffix, such as `_pset`, `_files`, `_sub`, `_pat`.
As steps themselves are the highest level entity and never used by other steps,
you donot need to name them with a special suffix.

::: group-tab

### XML

```xml
<parameterset name="my_pset">
  ...
</parameterset>
<fileset name="my_files">
  ...
</fileset>
<step name="mystep">
  <use>my_pset</use>
  <use>my_files</use>
</step>
```

### YAML
```yaml
parameterset:
    name: my_pset
    ...

fileset:
    name: my_files

step:
    name: mystep
    use:
        my_pset
        my_files
```
:::

Here are some suggestions for suffixes according to their type:

| Entity        | Suffix   |
|---------------|---------:|
| Fileset       | `_files` |
| Parameterset  |  `_pset` |
| Patterset     |   `_pat` |
| Substituteset |   `_sub` |

::::::

## Defining workflow steps

The only reason to define parameters in the first place is to *use* them in the execution of actions in your workflow steps.
To do this, the keyword `use`, referencing the name of a parameter set, is needed in the step definintion.
Actions are defined using the `do` keyword and should be standard shell
commands (including arguments).
Parameters can be referenced using the `$` prefix to its name and are replaced
by their respective value prior to the execution

::: group-tab

### XML
You can download the full definition so far as [hello_world.xml](files/hello_world.xml).
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
You can download the full definition so far as [hello_world.yaml](files/hello_world.yaml).
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

### Defining dependencies

A workflow only starts to make sense when you have more than one step and there
are dependencies between steps.
In JUBE you can define these dependencies with the **depend** keyword.

::: group-tab

### XML

```xml
<step name="initial_step">
...
</step>

<step name="dependent_step" depend="initial_step">
...
</step>
```

### YAML

```yaml
step:
  -name: initial_step
...
  -name: dependent_step
   depend: initial_step
...
```

:::

Steps in a workflow that depend on prior steps in the workflow are only
executed once the corresponding dependent step has executed successfully.

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

A JUBE workflow is initiated with the `run` command. At that time, the first
step will create workpackages based on the use parametersets.


::::::::::::::::::::::::::::::::::::: keypoints

- The different states of a JUBE workflow can be `running` and `completed`.
- A workflow is completed if all steps of the workflow are completed (even if some steps may have failed).
- A JUBE workflow is started with `jube run`.
- JUBE executes the workflow until the end, or until an asynchronous task is discovered.
- If JUBE exits with completion of asynchronous tasks still pending, the user needs to call `jube continue` to check for their completion and further running of dependent workflow steps until overall completion is achieved.
- The user can analyze output and print results at any time.
- Comma-separated values in parameter definition are tokenized and create individual workpackages.
  Using multiple comma-separated values for parameters enables easy creation of
  parameter spaces.
::::::::::::::::::::::::::::::::::::::::::::::::

