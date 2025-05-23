---
title: "Using templates"
teaching: 10
exercises: 0
---

::::::::::::::: questions
- How do I generate dynamic text-based input files for my workflow.
- How do I create batch scripts for by workflow.
:::::::::::::::::::::::::

::::::::::::::: objectives
- Create simple text-based templates generate files based on parameter settings.
::::::::::::::::::::::::::

::::::::::::::::: instructor
This episode introduces the simple text-based substitution system of JUBE to create text-based input files or batch scripts for submission to an HPC workload manager.
::::::::::::::::::::::::::::

In HPC workflows, oftentimes input files or batch scripts need to be created.
While some parts of these files are specific to the current workflow running, the files either have a static structure and/or even largely static content.
To handle this, JUBE provides a mechanism of generating text files based on search and replace strategy.

## Substitutions

At the heart of this process are *substitute sets*, which are a collection of individual *source* to *destination* replacements.
The replacements can either be based on static text or variables of parameters from any parameterset in use at the time of substitution.

:::::::::::::: group-tab
### XML
```xml
  <sub source="#STATIC_VAR_A#" dest="static_value" />
  <sub source="DYNAMIC_VAR_B#" dest="$param_b" />
```
### YAML
```yaml
  sub:
    - { source: "#STATIC_VAR_A#", dest: "static_value" }
    - { source: "#DYNAMIC_VAR_B#", dest: "$param_b" }
```
::::::::::::::::::::::::

Such substitutions need to be part of a substitute set that defines the input template as well as the output file.
These are defined by `iofile` clauses.
Each substitute set needs one of more such clauses.
Providing multiple `iofile` clauses will apply the same text replacements to all files listed.

:::::::::::::::: group-tab
### XML
```xml
<substituteset name="example_sub">
  <iofile int="template.in" out="generated_file">
  ...
</substituteset>
```
### YAML
```yaml
substituteset:
  - name: example_sub
    - iofile:
      - { in="template.in", out="generated_file" }
...
```
::::::::::::::::::::::::::

## File sets

To make the substitutions more independent, JUBE provides so called *file sets*.
These are a collection of clauses describing that either a file or directory should be *copied* or *linked* to the workpackage of a given step.

:::::::::::: caution
Linking files may reduce redundancy, especially if large read-only files and directories are involved in the workflow.
However, keep in mind that a link potentially creates a connection to a file path outside of the run directory, which may impact
concurrent steps and even separate runs.

To ensure reproducibility and consistency of workflow runs, the user should therefore check for potential side effects and employ a combination of copy and link commands that suits the workflow best.
::::::::::::::::::::

::::::::::::: group-tab
### XML
```xml
<fileset name="example_files">
  <copy>example_copy.dat</copy>
  <link>example_link.dat</copy>
</fileset>
```
### YAML
```yaml
fileset:
  - name: example_files
    copy:
      - example_copy.dat
    link:
      - example_link.dat
```
:::::::::::::::::::::::

:::::::: callout
Copy and link clauses can also specify additional attributes, such as `source_dir` and `target_dir` to specify a path to prefix the given filename with, `rel_path_ref` to indicate `external`reference (relative to the XML/YAML file) for the file (the default), or `internal` referencing the current working directory (to link to files of another step). See [the full description of the copy_tag for more details](https://apps.fz-juelich.de/jsc/jube/docu/glossar.html#term-copy_tag).
::::::::::::::::

::::::::::::::::::::::::: instructors
We end this without an exercise, as the next episode will include the generation of batch scripts using the above steps.
:::::::::::::::::::::::::::::::::::::



:::::::::::::::: keypoints
- Filesets let you easily copy templates into the workpackage space.
- Substitution sets perform simple text-based substitutions in templates.
- Using parameter values in substitution sets enables easy value manipulation during template generation.
::::::::::::::::::::::::::
