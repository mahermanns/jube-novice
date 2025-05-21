---
title: "Running an application"
teaching: 10
exercises: 2
---


:::::::::::::::::::::::::::::::::::::: questions

- How can you use JUBE to submit batch jobs on an HPC System?

::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: objectives

- Use the `done_file` attribute to deal with asynchronous commands.
- Automatically generate job scripts from templates.
- Automatically submit jobs on an HPC system.

::::::::::::::::::::::::::::::::::::::::::::::::

### Dealing with asynchronous commands

The commands in the `do` clauses so far, were all synchronous.
This means that the effect of the specified command is observable after the command completed.
For example a `mkdir -p my_dir` will return after the directory `my_dir` is created.
JUBE will continue execution of the next `do` clause defined in the step of complete the step.

HPC systems however are used differently than personal computers.
To ensure the resources are used fairly and efficiently, work is handed to a
scheduler to allocate resources for the job.
However, those resources may not be available right away.
The scheduler therefore does not block the command, but rather return to the user and handles
the job without further interaction of the user.

This means, however, the effect of the job script (a completed application run) will most likely not have materialized when the command return and JUBE shoud wait for it before continuing execution of the next `do` clause.
JUBE handles such asynchronous behavior via the `done_file` attribute of the `do` clause.
If specified, JUBE will not assume the successful execution of the specified command in the `do` clause for completion, but the existence of the the file specified in `done_file` as a side effect of the `do` command.

This way, JUBE does not need to understand the details of the asynchronous command specified, as long as the command eventually generates the file specified in `done_file`.
For HPC systems and batch jobs, this can be exploited by generating a file as part of the batch job, if the application ran successfully as part of the batch job.

```sh
$ nano batchscript.sh
```
```sh
....
srun ...
JUBE_ERR_CODE=$?
if [ $JUBE_ERR_CODE -ne 0 ]; then
    touch error
    exit $JUBE_ERR_CODE
fi
...
touch ready
```
In case of the SLURM scheduler, the corresponding `do` clause would then look like so.

::: group-tab

### XML
```xml
<do done_file="ready">sbatch batchscript.sh</do>
```
### YAML
```yaml
do:
 - { done_file: ready, _: sbatch batchscript.sh }
```
::::::::::::::


```output
######################################################################
# benchmark: GROMACS
# id: 24
#
# MD Simulation Workflow
######################################################################

Running workpackages (#=done, 0=wait, E=error):
########################################00000000000000000000 (  2/  3)

  |        stepname | all | open | wait | error | done |
  |-----------------|-----|------|------|-------|------|
  | prepare_sources |   1 |    0 |    0 |     0 |    1 |
  |           build |   1 |    0 |    0 |     0 |    1 |
  |             run |   1 |    0 |    1 |     0 |    0 |

>>>> Benchmark information and further useful commands:
>>>>       id: 24
>>>>   handle: jube_run
>>>>      dir: jube_run/000024
>>>> continue: jube continue jube_run --id 24
>>>>  analyse: jube analyse jube_run --id 24
>>>>   result: jube result jube_run --id 24
>>>>     info: jube info jube_run --id 24
>>>>      log: jube log jube_run --id 24
######################################################################
```

### Platform-independent workflows

JUBE offers batch job templates compatible with a variety of batch systems that utilize this mechanism.
The following code snippet demonstrates a block of shell commands that checks the return code of the most recently executed command.
If the return code is not 0 (zero), the action specified by the substitution parameter #FLAG_ERROR# is executed, and the script exits with the corresponding non-zero return code.
At the end of the batch script, the substitution parameter #FLAG# will generate the specified file.

```sh
JUBE_ERR_CODE=$?
if [ $JUBE_ERR_CODE -ne 0 ]; then
    #FLAG_ERROR#
    exit $JUBE_ERR_CODE
fi
...
#FLAG#
```
If we take a look into the corresponding substitution set, we can see te following configuration.
```xml
<sub source="#FLAG#" dest="touch $done_file" />
<sub source="#FLAG_ERROR#" dest="touch $error_file" />
```
and finally, the parameterset `executeset` in `platform.xml` contains the definition of
`done_file` and `error_file`.
```xml
<parameter name="done_file">ready</parameter>
<parameter name="error_file">error</parameter>
```




::::::::::::::::::::::::::::::::::::: keypoints

- JUBE provides a default job script template for different schedulers.
- JUBE allows for asynchronous execution of workflows using the `done_file`
  attribute.
- Asynchronous JUBE executions are dependent on the generation of files by the
  asynchronous task to continue.

::::::::::::::::::::::::::::::::::::::::::::::::




