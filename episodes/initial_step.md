---
title: "An initial workflow specification"
teaching: 10
exercises: 2
---

:::::::::::::::::::::::::::::::::::::: questions 

- How to use JUBE to automate the build process of a given HPC application?

::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: objectives

- Define an initial workflow.
- Create the benchmarks systems description automatically.

::::::::::::::::::::::::::::::::::::::::::::::::

## Writing an initial workflow

Mapping our experience with JUBE so far to the steps involved in compiling
GROMACS, we can define two different steps:

- download and unpack GROMACS sources
- building the application

::: challenge

Why don't we have more or less steps defined?

:::::: solution

If we would define all actions taken in the episode manually building GROMACS
in one step, we would always download the sources of GROMACS for each build.

If we would separate the *configure* and the *make* phases, those will seem
independent, yet there is always a single configure phase with each make phase.
Therefore, these two actions should be part of the same step.

::::::

:::


::: group-tab

### XML

Create an XML configuration for a GROMACS workflow with the name `gromacs.xml`
with the following configuration.
```sh
jube-workflow$ nano gromacs.xml
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<jube>
  <benchmark name="GROMACS" outpath="bench_run">
    <comment>MD Simulation Workflow</comment>
    <!-- further configuration goes here -->

    <step name="prepare_sources">
      <do>mkdir -p sources</do>
      <do>wget https://ftp.gromacs.org/gromacs/gromacs-2024.5.tar.gz</do>
      <do>tar xzf gromacs-2024.5.tar.gz</do>
    </step>
    <step name="build" depend="prepare_sources">
      <do>cmake -S prepare_sources/gromacs-2024.5/ -B build_gromacs</do>
      <do>cmake --build build_gromacs --parallel 12</do>
      <do>cmake --install build_gromacs --prefix install/gromacs-2024.5</do>
    </step>
  </benchmark>
</jube>
```
### YAML
```sh
jube-workflow$ nano gromacs.yaml
```
```yaml
name: GROMACS
outpath: jube_run
comment: MD Simulation Workflow

step:
  - name: prepare_sources
    do: 
      - wget https://ftp.gromacs.org/gromacs/gromacs-2024.5.tar.gz
      - tar xvzf gromacs-2024.5.tar.gz
  - name: build
    depend: prepare_sources
    do: 
      - cmake -S prepare_sources/gromacs-2024.5/ -B build_gromacs
      - cmake --build build_gromacs --parallel 12
      - cmake --install build_gromacs --prefix install/gromacs-2024.5
```
:::

After executing this workflow we can make several observations:

1. Some of the strings now hardcoded are part of a pattern that could be
   represented as a parameter instead.

2. Downloading and unpacking is done as part of the workflow into the
   workflow directory tree. Additional runs will download and unpack the sources again.


So let's address these issues one at a time.

## Reducing code-copy in the specification

In several locations, the version of GROMACS is referenced: in the source
archive, the unpacked source directory, and the installation path. If we'd
change the version in the future, we would have to change several locations in
the workflow configuration. We can reduce this by creating an appropriate
parameterset.

Edit your initial GROMACS workflow configuration and use parameters for key
variables in your workflow, to make this more flexible.

::: group-tab

### XML

```sh
jube-workflow$ nano gromacs.xml
```
```xml
<?xml version="1.0" encoding="UTF-8"?>
<jube>
  <benchmark name="GROMACS" outpath="bench_run">
    <comment>MD Simulation Workflow</comment>
    <!-- further configuration goes here -->
  </benchmark>
  <parameterset name="gromacs_pset">
    <parameter name="gromacs_version">2024.5</parameter>
    <parameter name="gromacs_sources">gromacs-$gromacs_version</parameterrameter>
    <parameter name="gromacs_archive">$gromacs_sources.tar.gz</parameter>
    <parameter name="gromacs_baseurl">https://ftp.gromacs.org/gromacs/</parameter>
    <parameter name="gromacs_build_dir">build_$gromacs_sources</parameter>
    <parameter name="gromacs_install_dir">install/$gromacs_sources</parameter>
  </parameterset>
  <step name="prepare_sources">
    <use>gromacs_pset</use>
    <do>wget $gromacs_baseurl/$gromacs_archive</do>
    <do>tar xzf $gromacs_archive</do>
  </step>
  <step name="build" depend="prepare_sources">
    <do>cmake -S prepare_sources/$gromacs_sources/ -B $gromacs_build_dir</do>
    <do>cmake --build $gromacs_build_dir --parallel 12</do>
    <do>cmake --install $gromacs_build_dir --prefix</do>
  </step>
</jube>
```

### YAML

```sh
jube-workflow$ nano gromacs.yaml
```
```yaml
name: GROMACS
outpath: jube_run
comment: MD Simulation Workflow

parameterset:
  name: gromacs_pset
  parameter:
    - { name: gromacs_version, _: 2024.5 }
    - { name: gromacs_sources, _: gromacs-$gromacs_version }
    - { name: gromacs_archive, _: $gromacs_sources.tar.gz }
    - { name: gromacs_baseurl, _: https://ftp.gromacs.org/gromacs/ }
    - { name: gromacs_build_dir, _: build_$gromacs_sources }
    - { name: gromacs_install_dir, _: install/$gromacs_sources }

step:
  - name: prepare_sources
    use: gromacs_pset
    do:
      - wget https://ftp.gromacs.org/gromacs/gromacs-2024.5.tar.gz
      - tar xvzf gromacs-2024.5.tar.gz
  - name: build
    depend: prepare_sources
    do:
      - make -S prepare_sources/$gromacs_sources/ -B $gromacs_build_dir
      - cmake --build $gromacs_build_dir --parallel 12
      - cmake --install $gromacs_build_dir --prefix $gromacs_install_dir
```
:::

## Breaking out of the workpackage sandbox

By default, JUBE will create a specific directory for each workpackage to
ensure that independent workpackages do not interfere with each other.
However, sometime it is helpful to break out of this safety net.

::: caution

The sandboxing of individual workflow runs and their workpackages is done for
good reason: to limit potential interactions among independent workpackages and
ensure consistency and reproducibility.

Only deviate from this when you have a very strong argument to do so.
:::

Next to workflow-specific parameters defined by the workflow itself, JUBE also
defines variables containing information about the current workflow run.
These variables can be referenced just as any parameter defined as part of a
parameterset.
One of these variables is `$jube_benchmark_home`, and it contains the absolute
path to the location of the workflow specification.

::: callout

Find a full list of internal variables set by JUBE in [the glossary of JUBE's documentation](https://apps.fz-juelich.de/jsc/jube/docu/glossar.html#term-jube_variables).

:::

Using this variable, we can now define the the installation path outside of the
directory structure referenced by `outpath`.
However, as any paths outside of JUBE's run directory tree will be accessed
(and potentially written to) by multiple workflow runs.
Therefore, you will need to take precautions not to overwrite installations
accidentially.


::: group-tab

### XML

```sh
jube-workflow$ nano gromacs.xml
```
```xml
<?xml version="1.0" encoding="UTF-8"?>
<jube>
  <benchmark name="GROMACS" outpath="bench_run">
    <comment>MD Simulation Workflow</comment>
    <!-- further configuration goes here -->
  </benchmark>
  <parameterset name="gromacs_pset">
    <parameter name="gromacs_version">2024.5</parameter>
    <parameter name="gromacs_sources">gromacs-$gromacs_version</parameterrameter>
    <parameter name="gromacs_archive">$gromacs_sources.tar.gz</parameter>
    <parameter name="gromacs_source_dir">$jube_benchmark_home/source</parameter>
    <parameter name="gromacs_baseurl">https://ftp.gromacs.org/gromacs/</parameter>
    <parameter name="gromacs_build_dir">build_$gromacs_sources</parameter>
    <parameter name="gromacs_install_dir">$jube_benchmark_home/$gromacs_sources</parameter>
  </parameterset>
  <step name="prepare_sources">
    <use>gromacs_pset</use>
    <do>mkdir -p $gromacs_source_dir</do>
    <do work_dir="$gromacs_source_dir">wget $gromacs_baseurl/$gromacs_archive</do>
    <do work_dir="$gromacs_source_dir">tar xzf $gromacs_archive</do>
  </step>
  <step name="build" depend="prepare_sources">
    <do>cmake -S $gromacs_source_dir/$gromacs_sources/ -B $gromacs_build_dir &gt; cmake_configure.log</do>
    <do>cmake --build $gromacs_build_dir --parallel 12 &gt; cmake_build.log</do>
    <do>cmake --install $gromacs_build_dir --prefix &gt; cmake_install.log</do>
    <do>touch $gromacs_install_dir/.install_complete</do>
  </step>
</jube>
```

### YAML

```sh
jube-workflow$ nano gromacs.yaml
```
```yaml
name: GROMACS
outpath: jube_run
comment: MD Simulation Workflow

parameterset:
  name: gromacs_pset
  parameter:
    - { name: gromacs_version, _: 2024.5 }
    - { name: gromacs_sources, _: gromacs-$gromacs_version }
    - { name: gromacs_archive, _: $gromacs_sources.tar.gz }
    - { name: gromacs_baseurl, _: https://ftp.gromacs.org/gromacs/ }
    - { name: gromacs_source_dir, _: $jube_benchmark_home/sources }
    - { name: gromacs_build_dir, _: build_$gromacs_sources }
    - { name: gromacs_install_dir, _: $jube_benchmark_home/$gromacs_sources }

step:
  - name: prepare_sources
    use: gromacs_pset
    do:
      - mkdir -p $gromacs_source_dir
      - { work_dir: $gromacs_source_dir, _: wget https://ftp.gromacs.org/gromacs/$gromacs_archive }
      - { work_dir: $gromacs__dir, _:tar xvzf $gromacs_archive }
  - name: build
    depend: prepare_sources
    do:
      - cmake -S prepare_sources/$gromacs_sources/ -B $gromacs_build_dir > cmake_configure.log
      - cmake --build $gromacs_build_dir --parallel 12 > cmake_build.log
      - cmake --install $gromacs_build_dir --prefix $gromacs_install_dir > cmake_install.log
      - touch $gromacs_install_dir/.install_complete
```
:::

::::::::::::::::::::::::::::::::::::: keypoints

- Group actions that belong together and have a 1:1 relation ship in a single
  step.
- Basing parameter values on other parameter values can help code copy and
  increase flexibility and maintainability of your workflow.
- You can generate build files from templates using dynamic values from
  parameter sets.

::::::::::::::::::::::::::::::::::::::::::::::::



