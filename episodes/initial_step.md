---
title: "An initial workflow step"
teaching: 10
exercises: 2
---

:::::::::::::::::::::::::::::::::::::: questions 

- How to use JUBE to automate the build process of a given HPC application?

::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: objectives

- Define an initial 
- Create the benchmarks systems description automatically.

::::::::::::::::::::::::::::::::::::::::::::::::

## Defining steps

:::::::::::::::::::::::::::: callout

## Naming conventions

Several entities defined in the specification are later used by its name in the step definitions.
To make it easier to understand more complex configurations, it is good practice to encode the type of the entity into its name, for example as a suffix, such as `_pset`, `_files`, `_sub`, `_pat`.

:::::::::::::::::::::::::::::: spoiler
### XML example
```xml
<parameterset name="my_pset">
  ...
</parameterset>
<fileset name="my_files">
  ...
</fileset>
<step name="my_step">
  <use>my_pset</use>
  <use>my_files</use>
</step>
```
::::::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::: spoiler
### YAML example
```xml
parameterset:
    name: my_pset
    ...

fileset:
    name: my_files

step:
    use:
        my_pset
        my_files
```
::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::


::::::::::::::::::::::::::::::::::::: keypoints

- You can generate build files from templates using dynamic values from
  parameter sets.

::::::::::::::::::::::::::::::::::::::::::::::::



