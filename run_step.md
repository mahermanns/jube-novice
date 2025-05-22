---
title: "Running an application"
teaching: 10
exercises: 5
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

Any workpackage with a pending `done_file` will be listed under `wait` in the table of open tasks.
To try to advance any waiting workpackage the user needs to needs the `continue` command, as listed in the output given by JUBE.

```sh
$ jube continue jube_run --id 24
```
```output
```output
######################################################################
# benchmark: GROMACS
# id: 24
#
# MD Simulation Workflow
######################################################################

Running workpackages (#=done, 0=wait, E=error):
############################################################ (  3/  3)

  |        stepname | all | open | wait | error | done |
  |-----------------|-----|------|------|-------|------|
  | prepare_sources |   1 |    0 |    0 |     0 |    1 |
  |           build |   1 |    0 |    0 |     0 |    1 |
  |             run |   1 |    0 |    0 |     0 |    1 |

>>>> Benchmark information and further useful commands:
>>>>       id: 24
>>>>   handle: jube_run
>>>>      dir: jube_run/000024
>>>>  analyse: jube analyse jube_run --id 24
>>>>   result: jube result jube_run --id 24
>>>>     info: jube info jube_run --id 24
>>>>      log: jube log jube_run --id 24
######################################################################
```

### Platform-independent workflows

JUBE offers batch job templates compatible with a variety of batch systems that utilize this mechanism.
These files are available in the JUBE installation directory unter `share/jube/platform`, with subdirectories per scheduler.
Provided with version 2.7.1 of JUBE are files prepared for **LSF**, **Moab**, **PBS**, and **SLURM**.

In each of these directories JUBE provides at least the two files `platform.xml` and `submit.job.in`.
The former is a collection of parameter sets, file sets, and substitution sets.
The latter is a template for the corresponding scheduler.

::::::::::::::::: callout
By specifying the path corresponding to your scheduler as part of the environment variable `JUBE_INCLUDE_PATH`, you can make both files readily available inside your workflow scripts.
```sh
$ export JUBE_INCLUDE_PATH=/path/to/your/jube/base/share/jube/platform/slurm:$JUBE_INCLUDE_PATH
```
:::::::::::::::::::::::::

Taking a look into the batch script template reveils the semingly automatic handling of generating either an `error` or a `ready` file as part of the batch job.

```sh
...
JUBE_ERR_CODE=$?
if [ $JUBE_ERR_CODE -ne 0 ]; then
    #FLAG_ERROR#
    exit $JUBE_ERR_CODE
fi
...
#FLAG#
```
If the return code is not 0 (zero), the action specified by the substitution parameter #FLAG_ERROR# is executed, and the script exits with the corresponding non-zero return code.
At the end of the batch script, the substitution parameter #FLAG# will generate the specified file.

If we take a look into the corresponding substitution set in `platform.xml`, we can see te following configuration.
```xml
<sub source="#FLAG#" dest="touch $done_file" />
<sub source="#FLAG_ERROR#" dest="touch $error_file" />
```
as well as the corresponding the parameterset `executeset` in `platform.xml` contains the definition of
`done_file` and `error_file`.
```xml
<parameter name="done_file">ready</parameter>
<parameter name="error_file">error</parameter>
```

Using these pre-defined sets in `platform.xml` in combination with the batch script template, we can easily create the *run* step for GROMACS.

:::::::::::: group-tab
### XML
```xml
<parameterset name="gromacs_run_pset">
    <parameter name="gromacs_data_basedir">${jube_benchmark_home}/data</parameter>
    <parameter name="gromacs_data_file_prefix">MD_${gromacs_domain_size_uc}_WATER</parameter>
    <parameter name="gromacs_data_file">${gromacs_data_basedir}/${gromacs_data_file_prefix}.tpr</parameter>
    <!--  default data size is 'small' -->
    <parameter name="gromacs_data_size">small</parameter>
    <!--  choose a specific data size via tag -->
    <parameter name="gromacs_data_size" tag="small">small</parameter>
    <parameter name="gromacs_data_size" tag="medium">medium</parameter>
    <parameter name="gromacs_data_size" tag="large">large</parameter>
    <!--  select the domain size via data size value -->
    <parameter name="gromacs_domain_size" mode="python">
        {
            "small":   "5nm",
            "medium": "10nm",
            "large" : "15nm"
        }.get("${gromacs_data_size}")
    </parameter>
    <parameter name="gromacs_domain_size_uc"  mode="python">"${gromacs_domain_size}".upper()</parameter>
    <parameter name="gromacs_timesteps">10000</parameter>
</parameterset>

<parameterset name="gromacs_system_pset" init_with="platform.xml:systemParameter">
    <parameter name="nodes">1</parameter>
    <parameter name="taskspernode">1</parameter>
    <parameter name="threadspertask">24</parameter>
</parameterset>

<parameterset name="gromacs_exec_pset" init_with="platform.xml:executeset">
</parameterset>

<fileset name="gromacs_input_files">
    <copy>$gromacs_data_file</copy>
</fileset>

<substituteset name="gromacs_exec_sub" init_with="platform.xml:executesub">
    <sub source="#EXECUTABLE#" dest="${gromacs_install_dir}/bin/gmx_mpi" />
    <sub source="#ARGS_EXECUTABLE#" dest="mdrun -deffnm ${gromacs_data_file_prefix} -nsteps ${gromacs_timesteps}" />
</substituteset>
```
### YAML
```yaml
parameterset:
  ...
  - name: gromacs_run_pset
    parameter:
      - { name: gromacs_data_basedir, _: '${jube_benchmark_home}/data' }
      - { name: gromacs_data_file_prefix, _: 'MD_${gromacs_domain_size_uc}_WATER' }
      - { name: gromacs_data_file, _: '${gromacs_data_basedir}/${gromacs_data_file_prefix}.tpr' }
      # default data size is 'small'
      - { name: gromacs_data_size, _: "small" }
      # choose a specific data size via tag
      - { name: gromacs_data_size, tag: small, _: "small" }
      - { name: gromacs_data_size, tag: medium, _: "medium" }
      - { name: gromacs_data_size, tag: large, _: "large" }
      # select the domain size via data size value
      - { name: gromacs_domain_size, mode: python, _: '
          {
              "small":   "5nm",
              "medium": "10nm",
              "large" : "15nm"
          }.get("${gromacs_data_size}")'
        }
      - { name: gromacs_domain_size_uc, mode: python, _: '"${gromacs_domain_size}".upper()' }
      - { name: gromacs_timesteps, _: 10000 }
  - name: gromacs_system_pset
    init_with: platform.xml:systemParameter
    parameter:
      - { name: nodes, _: 1 }
      - { name: taskspernode, _: 1 }
      - { name: threadspertask, _: 24 }
  - name: gromacs_exec_pset
    init_with: platform.xml:executeset
fileset:
  - name: gromacs_input_files
    copy:
      - $gromacs_data_file

substituteset:
  - name: gromacs_exec_sub
    init_with: platform.xml:executesub
    sub:
      - { source: "#EXECUTABLE#", dest: "${gromacs_install_dir}/bin/gmx_mpi" }
      - { source: "#ARGS_EXECUTABLE#", dest: "mdrun -deffnm ${gromacs_data_file_prefix} -nsteps ${gromacs_timesteps}" }

step:
...
- name: run
    depend: build
    use:
      - gromacs_run_pset
      - gromacs_exec_pset
      - gromacs_system_pset
      - gromacs_input_files
      - { from: platform.xml, _: jobfiles }
      - gromacs_exec_sub
    do:
      - { done_file: $done_file, _: "${submit} ${submit_script}" }
```
::::::::::::::::::::::

Using the indirecttion of the corresponding sets in the `platform.xml` we now have a workflow specification that does not in itself reference any specific scheduler, as that is handled via the `JUBE_INCLUDE_PATH` from the shell JUBE is executed in.

:::::::::::::::: callout
Checkout further substitution patterns available in the batch script template to identify parameters you can use to add commands to your batch script.

Note that those scripts and definitions are only for your convenience. If they don't fit your need, you can adapt them to your needs or fully rely on self-provided configurations.
::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: keypoints

- JUBE provides a default job script template for different schedulers.
- JUBE allows for asynchronous execution of workflows using the `done_file`
  attribute.
- Asynchronous JUBE executions are dependent on the generation of files by the
  asynchronous task to continue.

::::::::::::::::::::::::::::::::::::::::::::::::




