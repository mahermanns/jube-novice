---
title: "Influencing parameters with tags"
teaching: 10
exercises: 2
---

:::::::::::::::::::::::::::::::::::::: questions

- How can a workflow be influenced from the command line without changing the
  workflow specification files?

::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: objectives

- Use tags to influence configuration values.
- Run different workflows with the same configuration.

::::::::::::::::::::::::::::::::::::::::::::::::

Now that we transferred our initial workflow to a JUBE configuration, we would
like to extend this to work with other versions of GROMACS as well.

Fortunately, all names in the URL and archive follow the same convension, and
we have already separated the values involved into the proper parameters.
So now, we can modify our GROMACS workflow to use a different version of the
software.

```sh
$ nano gromacs.xml
```

::: group-tab

### XML
```xml
<parameter name="gromacs_version">2024.1</parameter>
```

### YAML
```yaml
  parameter: {name: gromacs_version, _: 2024.1}
```

:::

When we run our workflow again, it will download and build the 2024.1 version
of the GROMACS software.

```sh
$ jube run gromacs.xml
```

::: instructor
You can demonstrate the workflow run and showcase that the new archive was
downloaded, and the software eventually installed in a different directory.
:::

Now, we might not want to modify our configuration every time directly if we
want to compare runs of two different versions, or comment in or our specific
lines in our parametersets.

## Uniqueness of parameters

Parameters in JUBE have to be globally unique.
That means now two different parametersets are allowed to define a parameter of
the same name.

However, if several parameter definitions for the same name exist, the last
definition will determine the parameter's value during the workflow run.

The following specification will result in the parameter `var` being assigned
the value `two`.

:::::: group-tab

### XML
```xml
<parameter name="var">one</parameter>
<parameter name="var">two</parameter>
```

### YAML
```yaml
  parameter: {name: var, _: one}
  parameter: {name: var, _: two}
```

::::::::::::::::

## Visibility of configuration entities

JUBE allows the attribute `tag` for any part of the configuration.
Tagged parts of the configuration are either retained in or removed from the
workflow specification during a run, depending on tags for that
run specified on the command line.

In the presence of tags,

1. Any entity without a `tag` attribute will be retained in the workflow.
2. Any entity with a `tag` attribute and the value of the attribute defined,
   will be retained in the workflow.
3. Any entity with a `tag` attribute and the value of the attribute not
   defined, will be removed in the workflow.

Tag attributes are also allowed to be comma-separated lists of tag names.
A specification of `!` (exclamation mark) in front of a tag name will invert
rules 2 and 3 above (*not* specification).

The *not* (`!`) specification has higher precedence than the normal tag
specification. Consider the following parameter definitions:

::: group-tab

### XML
```xml
<parameter name="var" tag="a,!b,c">...</parameter>
```
### YAML
```yaml
  parameter: {name: var, tag: "a,!b,c", _: ...}
```
:::::::::::::

When specified with `--tag a b`, the parameter will removed for that run, as
the `!b` takes higher precedence over the `a`.


::: challenge

Define the `gromacs_version` parameter such that per default the version 2024.5
is chosen, but in the presense of the tag `v2024.1` the version 2024.1.

:::::: solution

:::::::::: group-tab
### XML
```xml
<parameter name="gromacs_version">2024.5</parameter>
<parameter name="gromacs_version" tag="v2024.1">2024.1>/parameter>
```
### YAML
```yaml
  parameter: {name: gromacs_version, _: 2024.5}
  parameter: {name: gromacs_version, tag: v2024.1, _: 2024.1}
```
::::::::::::::::::::

:::::::::::::::


:::::::::::::


### Using tags to enable a forced rebuild

Using the `tag` functionality, we can add a second definintion of the `build_gromacs` parameter with a *rebuild* tag to the workflow specification to
force JUBE to rebuild of GROMACS just by adding the *rebuild* tag on the command line when running a workflow.

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
    <parameter name="gromacs_install_indicator">.install_complete</parameter>
    <parameter name="build_gromacs" mode="shell">if [ -e "$gromacs_install_dir/$gromacs_install_indicator" ]; then echo false; else echo true; fi</parameter>
    <parameter name="build_gromacs" tag="rebuild">true</parameter>
  </parameterset>
  <step name="prepare_sources">
    <use>gromacs_pset</use>
    <do>mkdir -p $gromacs_source_dir</do>
    <do work_dir="$gromacs_source_dir">wget $gromacs_baseurl/$gromacs_archive</do>
    <do work_dir="$gromacs_source_dir">tar xzf $gromacs_archive</do>
  </step>
  <step name="build" depend="prepare_sources">
    <do active="$build_gromacs">if [ -d "$gromacs_install_dir" ]; then rm -rf "$gromacs_install_dir"; fi</do>
    <do active="$build_gromacs">cmake -S $gromacs_source_dir/$gromacs_sources/ -B $gromacs_build_dir &gt; cmake_configure.log</do>
    <do active="$build_gromacs">cmake --build $gromacs_build_dir --parallel 12 &gt; cmake_build.log</do>
    <do active="$build_gromacs">cmake --install $gromacs_build_dir --prefix &gt; cmake_install.log</do>
    <do active="$build_gromacs">mkdir -p $gromacs_install_dir/share/ &amp;&amp; cp cmake_configure.log cmake_build.log cmake_install.log $gromacs_install_dir/share</do>
    <do active="$build_gromacs">touch "$gromacs_install_dir/$gromacs_install_indicator"</do>
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
...
    - { name: build_gromacs, mode: shell, _: if [ -e "$gromacs_install_dir/$gromacs_install_indicator" ]; then echo false; else echo true; fi }
    - { name: build_gromacs tag: rebuild, _: true }

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
      - { active: $build_gromacs _: if [ -d "$gromacs_install_dir" ]; then rm -rf "$gromacs_install_dir"; fi }
      - { active: $build_gromacs _: cmake -S prepare_sources/$gromacs_sources/ -B $gromacs_build_dir > cmake_configure.log }
      - { active: $build_gromacs _: cmake --build $gromacs_build_dir --parallel 12 > cmake_build.log }
      - { active: $build_gromacs _: cmake --install $gromacs_build_dir --prefix $gromacs_install_dir > cmake_install.log" }
      - { active: $build_gromacs _: mkdir -p $gromacs_install_dir/share/ &amp;&amp; cp cmake_configure.log cmake_build.log cmake_install.log $gromacs_install_dir/share }
      - { active: $build_gromacs _: touch "$gromacs_install_dir/$gromacs_install_indicator" }
```
:::


::::::::::::::::::::::::::::::::::::: keypoints

- Use tags to include or exclude specific parts of a workflow.

::::::::::::::::::::::::::::::::::::::::::::::::

