---
title: "Output analysis and presentation"
teaching: 10
exercises: 10
---

:::::::::::::::::::::::::::::::::::::: questions

- How can you retrieve output from your HPC application and workflow?
- How to generate tables and CSV output from JUBE workflow parameters?

::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: objectives

- Generate regular expression patterns to analyse the output of a step.
- Generate a basic table from benchmark parameters and output.

::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::: prereq
If you cannot execute GROMACS via the generated batch script to generate an output, you can down load an example error log [job.err](episodes/files/job.err) and place it in the work directory of your *run* stage (`jube_run/<id>/000002_run/work/`).
Furthermore you need to let JUBE know that the execution was successful with the following command executed in the same directory.
```sh
jube_run/<id>/000002_run/work/$ touch ready
```
Then continue the workflow to complete it.
:::::::::::::::::::::::::::::

## Analyzing workflow output

Next to parameters, which are evaluated at use time in the steps of a workflow, another way to automatically obtain and store information about a workflow execution is to analyze the output generated during execution.
JUBE provides an easy way for regular expressions to parse workflow output and store values of matched patterns in variables to be included in later result tables.

Patterns are defined as part of pattern sets that can be applied to specific files during analysis.
As with parameters, the names of patterns need to be unique for the whole workflow.
When included in result tables, their names can be shortened or changed completely to better fit the generated table.


:::::::::::::: challenge
For the workflow defined, we have two steps with potentially valuable information: *build* and *run*.
Check the outputs in the corresponding workpackages and identify interesting output files to parse for meaningful data.
::::::::::::::::::::::::
::::::::::::::: hint
- The [`cmake_configure.log`](episodes/files/cmake_configure.log) in the *build* workpackage contains output from CMake that provides some insight on which libraries were found and used for the build.
- The [`stdout`](episodes/files/stdout) in the *run* workpackage contains the output of the job submission command
- The [`job.err`](episodes/files/job.err) in the *run* workpackage contains the GROMACS output.

::::::::::::::::::::::::

Patterns are defines as [regular expressions][carpentries-regex] as part of a pattern set.
When we take the following snippets from [`cmake_configure.log`](episodes/files/cmake_configure.log)

```output
-- The C compiler identification is IntelLLVM 2024.2.0
-- The CXX compiler identification is IntelLLVM 2024.2.0
...
-- Detected best SIMD instructions for this CPU - AVX_512
-- Enabling 512-bit AVX-512 SIMD instructions using CXX flags:  -march=skylake-avx512
...
-- Using external FFT library - Intel MKL
...
```

we can already identify some interesting information that are worth extracting.

:::::::::::: group-tab
### XML
```xml
    <patternset name="cmake_configure_patterns">
      <pattern name="cmake_c_compiler_id">The C compiler identification is ([^ ]*)</pattern>
      <pattern name="cmake_c_compiler_version">The C compiler identification is [^ ]* ([^ ]*)$</pattern>
      <pattern name="cmake_cxx_compiler_id">The CXX compiler identification is ${jube_pat_wrd}</pattern>
      <pattern name="cmake_cxx_compiler_version">The CXX compiler identification is ${jube_pat_nwrd} ${jube_pat_wrd}</pattern>
      <pattern name="SIMD_detected">-- Detected best SIMD instructions for this CPU - ${jube_pat_wrd}</pattern>
      <pattern name="SIMD_flags" dotall="false">-- Enabling.*SIMD instructions using CXX flags:${jube_pat_bl}(.*)</pattern>
      <pattern name="FFT_detected" dotall="false">-- Using external FFT library - (.*)$</pattern>
    </patternset>
```
### YAML
```yaml
patternset:
  - name: cmake_configure_patterns
    pattern:
      - { name: cmake_c_compiler_id, _: "The C compiler identification is ([^ ]*)" }
      - { name: cmake_c_compiler_version, _: "The C compiler identification is [^ ]* ([^ ]*)$" }
      - { name: cmake_cxx_compiler_id, _: "The CXX compiler identification is ${jube_pat_wrd}" }
      - { name: cmake_cxx_compiler_version, _: "The CXX compiler identification is ${jube_pat_nwrd} ${jube_pat_wrd}" }
      - { name: SIMD_detected, _: "-- Detected best SIMD instructions for this CPU - ${jube_pat_wrd}" }
      - { name: SIMD_flags, dotall: false, _: "Enabling.*SIMD instructions using CXX flags:${jube_pat_bl}(.*)" }
      - { name: FFT_detected, dotall: false, _: "Using external FFT library - (.*)$" }
  - name: gromacs_output_patterns
```
::::::::::::::::::::::

Each pattern can contain an arbitrary amount of wildcards, but must contain exactly one *matching* operator `()`, which defines the value of the pattern.
JUBE also defines [several patterns](https://apps.fz-juelich.de/jsc/jube/docu/glossar.html#term-jube_pattern) for common elementar types such as numbers and individual words:

- `$jube_pat_int`: integer number w/ matching operator
- `$jube_pat_nint`: integer number w/o matchin operator
- `$jube_pat_fp`: floating point number w/ matching operator
- `$jube_pat_nfp`: floating point number w/o matching operator
- `$jube_pat_wrd`: word w/ matching operator
- `$jube_pat_nwrd`: word w/o matching operator
- `$jube_pat_bl`: blank space (variable length) w/o matching operator


::::::::::::::: callout
While all **steps** of a JUBE workflow need to be defined before the workflow is started and cannot be altered after that, patterns and result tables can be updated while the workflow is active and even after a workflow has completed.
This enables an iterative approach for defining patterns and result tables for previously unknown output.
:::::::::::::::::::::::

To use the defined patterns, JUBE needs an `analyser` specifying which patterns to use for which file or which patterns to use globally.

::::::::::::::::: group-tab
### XML
```xml
<analyser name="gromacs_analyser">
    <analyse step="build">
        <file use="cmake_configure_patterns">$gromacs_configure_log</file>
    </analyse>
</analyser>
```
### YAML
```yaml
analyser:
  - name: gromacs_analyser
    analyse:
      - step: build
        file:
          - { use: cmake_configure_patterns, _: $gromacs_configure_log }
result:
```
:::::::::::::::::::::::::::

We can then start and test the analyser with the `analyse` command.
However, as we modified the workflow configuration by adding new patterns and analyser definitions, we need to tell JUBE to update its information about the workflow. This can be done with the `-u` (for update) argument followed by the updated workflow specification.

:::::::::::::: group-tab
### XML
```sh
$ jube analyse -u gromacs.xml jube_run --id 35
```
### YAML
```sh
$ jube analyse -u gromacs.yaml jube_run --id 35
```
:::::::::::::::::::::::::
```output
######################################################################
# Analyse benchmark "GROMACS" id: 35
######################################################################
>>> Start analyse
>>> Analyse finished
>>> Analyse data storage: jube_run/000035/analyse.xml
######################################################################
```

Without the definition of a result table, we cannot visualise this directly, but we can investigate the data storage given in the output and check which patterns were matched for which workpackage.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<analyse>
  <analyser name="gromacs_analyser">
    <step name="build">
      <workpackage id="1">
        <pattern name="cmake_c_compiler_id" type="string">IntelLLVM</pattern>
        <pattern name="cmake_c_compiler_id_first" type="string">IntelLLVM</pattern>
        <pattern name="cmake_c_compiler_id_cnt" type="int">1</pattern>
        <pattern name="cmake_c_compiler_id_last" type="string">IntelLLVM</pattern>
        <pattern name="cmake_c_compiler_version" type="string">2024.2.0</pattern>
        <pattern name="cmake_c_compiler_version_first" type="string">2024.2.0</pattern>
        <pattern name="cmake_c_compiler_version_cnt" type="int">1</pattern>
        <pattern name="cmake_c_compiler_version_last" type="string">2024.2.0</pattern>
        <pattern name="cmake_cxx_compiler_id" type="string">IntelLLVM</pattern>
        <pattern name="cmake_cxx_compiler_id_first" type="string">IntelLLVM</pattern>
        <pattern name="cmake_cxx_compiler_id_cnt" type="int">1</pattern>
        <pattern name="cmake_cxx_compiler_id_last" type="string">IntelLLVM</pattern>
        <pattern name="cmake_cxx_compiler_version" type="string">2024.2.0</pattern>
        <pattern name="cmake_cxx_compiler_version_first" type="string">2024.2.0</pattern>
        <pattern name="cmake_cxx_compiler_version_cnt" type="int">1</pattern>
        <pattern name="cmake_cxx_compiler_version_last" type="string">2024.2.0</pattern>
        <pattern name="SIMD_detected" type="string">AVX_512</pattern>
        <pattern name="SIMD_detected_first" type="string">AVX_512</pattern>
        <pattern name="SIMD_detected_cnt" type="int">1</pattern>
        <pattern name="SIMD_detected_last" type="string">AVX_512</pattern>
        <pattern name="SIMD_flags" type="string">-march=skylake-avx512</pattern>
        <pattern name="SIMD_flags_first" type="string">-march=skylake-avx512</pattern>
        <pattern name="SIMD_flags_cnt" type="int">1</pattern>
        <pattern name="SIMD_flags_last" type="string">-march=skylake-avx512</pattern>
        <pattern name="FFT_detected" type="string">Intel MKL</pattern>
        <pattern name="FFT_detected_first" type="string">Intel MKL</pattern>
        <pattern name="FFT_detected_cnt" type="int">1</pattern>
        <pattern name="FFT_detected_last" type="string">Intel MKL</pattern>
      </workpackage>
    </step>
  </analyser>
</analyse>
```

JUBE also tracks multiple matches per pattern and tracks it in "shadow" patterns with additional suffixes.
You can find a more details description [in the JUBE Glossary under 'statistical values'](https://apps.fz-juelich.de/jsc/jube/docu/glossar.html#term-statistical_values).
For numerical statistics (e.g., min, max, avg, std) the pattern needs to be of **type** `int` or `float`.
Also, a **unit** specified as a string can be stored with a pattern.


## Generating a result table

With our first successful matches, we can now define our first result table for build-related information in a *result* specification.

:::::::::::::::::::: group-tab
### XML
```xml
<result>
  <use>gromacs_analyser</use>
  <table name="gromacs_build" style="pretty">
        <column title="compiler">cmake_cxx_compiler_id</column>
        <column title="compiler_version">cmake_cxx_compiler_version</column>
        <column title="SIMD">SIMD_detected</column>
        <column title="FFT">FFT_detected</column>
    </table>
  </table>
</result>
```
### YAML
```yaml
result:
  use: gromacs_analyser
  table:
    - name: gromacs_build
      style: pretty
      column:
        - { title: "compiler", _: cmake_cxx_compiler_id }
        - { title: "compiler version", _:  cmake_cxx_compiler_version }
        - { title: "SIMD", _: SIMD_detected }
        - { title: "FFT", _: FFT_detected }
```
::::::::::::::::::::::::::::::
```output
gromacs_build:
|  compiler | compiler version |    SIMD |       FFT |
|-----------|------------------|---------|-----------|
| IntelLLVM |         2024.2.0 | AVX_512 | Intel MKL |
```


:::::::::::: challenge
Add an additional patternset, analyser and result definition to generate an additional table similar to the following:
```output
gromacs_run:
| wp | gromacs_core_time[s] | gromacs_wall_time[s] | gromacs_core_perf[ns/day] | gromacs_wall_perf[hours/ns] |
|----|----------------------|----------------------|---------------------------|-----------------------------|
|  2 |               89.664 |                3.736 |                    89.664 |                       3.736 |
```
The `wp` column references a JUBE variable identifying the *workpackage* of the row. This makes it easier to identify the right directory of the workpackage in case multiple run steps were executed.
::::::::::::::::::::::
:::::::::::: solution
:::: group-tab
### XML
```xml
<patternset name="cmake_configure_patterns">
    <pattern name="cmake_c_compiler_id">The C compiler identification is ([^ ]*)</pattern>
    <pattern name="cmake_c_compiler_version">The C compiler identification is [^ ]* ([^ ]*)$</pattern>
    <pattern name="cmake_cxx_compiler_id">The CXX compiler identification is ${jube_pat_wrd}</pattern>
    <pattern name="cmake_cxx_compiler_version">The CXX compiler identification is ${jube_pat_nwrd} ${jube_pat_wrd}</pattern>
    <pattern name="SIMD_detected">-- Detected best SIMD instructions for this CPU - ${jube_pat_wrd}</pattern>
    <pattern name="SIMD_flags" dotall="false">-- Enabling.*SIMD instructions using CXX flags:$jube_pat_bl(.*)</pattern>
    <pattern name="FFT_detected" dotall="false">-- Using external FFT library - (.*)$</pattern>
</patternset>
<patternset name="gromacs_output_patterns">
    <pattern name="gromacs_num_procs" unit="s">Using ${jube_pat_int} MPI proc.*</pattern>
    <pattern name="gromacs_num_threads" unit="s">Using ${jube_pat_int} OpenMP thread.*</pattern>
    <pattern name="gromacs_core_time" unit="s">Time:\s*${jube_pat_fp}</pattern>
    <pattern name="gromacs_wall_time" unit="s">Time:\s*${jube_pat_nfp}\s*${jube_pat_fp}</pattern>
    <pattern name="gromacs_core_perf" unit="ns/day">Performance:\s*${jube_pat_fp}</pattern>
    <pattern name="gromacs_wall_perf" unit="hours/ns">Performance:\s*${jube_pat_nfp}\s*${jube_pat_fp}</pattern>
</patternset>
<analyser name="gromacs_analyser">
    <analyse step="build">
        <file use="cmake_configure_patterns">$gromacs_configure_log</file>
    </analyse>
    <analyse step="run">
        <file use="gromacs_output_patterns">$errlogfile</file>
    </analyse>
</analyser>

<result>
    <use>gromacs_analyser</use>
    <table name="gromacs_build" style="pretty">
        <column title="compiler">cmake_cxx_compiler_id</column>
        <column title="compiler_version">cmake_cxx_compiler_version</column>
        <column title="SIMD">SIMD_detected</column>
        <column title="FFT">FFT_detected</column>
    </table>
    <table name="gromacs_run" style="pretty">
        <column title="wp">jube_wp_id</column>
        <column>gromacs_core_time</column>
        <column>gromacs_wall_time</column>
        <column>gromacs_core_perf</column>
        <column>gromacs_wall_perf</column>
    </table>
</result>
```
### YAML
```yaml
patternset:
  - name: cmake_configure_patterns
    pattern:
      - { name: cmake_c_compiler_id, _: "The C compiler identification is ([^ ]*)" }
      - { name: cmake_c_compiler_version, _: "The C compiler identification is [^ ]* ([^ ]*)$" }
      - { name: cmake_cxx_compiler_id, _: "The CXX compiler identification is ${jube_pat_wrd}" }
      - { name: cmake_cxx_compiler_version, _: "The CXX compiler identification is ${jube_pat_nwrd} ${jube_pat_wrd}" }
      - { name: SIMD_detected, _: "-- Detected best SIMD instructions for this CPU - ${jube_pat_wrd}" }
      - { name: SIMD_flags, dotall: false, _: "Enabling.*SIMD instructions using CXX flags:${jube_pat_bl}(.*)" }
      - { name: FFT_detected, dotall: false, _: "Using external FFT library - (.*)$" }
  - name: gromacs_output_patterns
    pattern:
      - { name: gromacs_num_procs, unit: "s", _: "Using ${jube_pat_int} MPI proc.*" }
      - { name: gromacs_num_threads, unit: "s", _: "Using ${jube_pat_int} OpenMP thread.*" }
      - { name: gromacs_core_time, unit: "s", _: "Time:\\s*${jube_pat_fp}" }
      - { name: gromacs_wall_time, unit: "s", _: "Time:\\s*${jube_pat_nfp}\\s*${jube_pat_fp}" }
      - { name: gromacs_core_perf, unit: "ns/day",  _: "Performance:\\s*${jube_pat_fp}" }
      - { name: gromacs_wall_perf, unit: "hours/ns", _: "Performance:\\s*${jube_pat_nfp}\\s*${jube_pat_fp}" }

analyser:
  - name: gromacs_analyser
    analyse:
      - step: build
        file:
          - { use: cmake_configure_patterns, _: $gromacs_configure_log }
      - step: run
        file:
          - { use: gromacs_output_patterns, _: $errlogfile }
result:
  use: gromacs_analyser
  table:
    - name: gromacs_build
      style: pretty
      column:
        - { title: "compiler", _: cmake_cxx_compiler_id }
        - { title: "compiler version", _:  cmake_cxx_compiler_version }
        - { title: SIMD support", _: SIMD_}
    - name: gromacs_run
      style: pretty
      column:
        - { title: "wp", _: jube_wp_id }
        - gromacs_core_time
        - gromacs_wall_time
        - gromacs_core_perf
        - gromacs_wall_perf

```
::::::::::::::
:::::::::::::::::::::

::::::::::::::::: callout
Each table has a separate file in the `result/` directory with its *name* as is name and the extension `.dat`.

Tables can be either *pretty* printed or in *CSV* (comma-separated values) format. The latter being the default type.
:::::::::::::::::::::::::


::::::::::::::::::::::::::::::::::::: keypoints

- JUBE allows for definition of patterns to retrieve values from a workflow
  step.
- Patterns can be defined as regular expressions or Python expressions.
- JUBE allows for the generate of pretty-printed tables and CSV format.

::::::::::::::::::::::::::::::::::::::::::::::::



