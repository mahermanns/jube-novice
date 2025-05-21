---
title: "Spanning parameter spaces"
teaching: 10
exercises: 10
---

:::::::::::::::::::::::::::::::::::::: questions

- How to test multiple parameters in a single workflow?
- How to control multi-dimensional parameter spaces?

::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: objectives

- Create multiple workpackages with comma-separated lists.
- Control parameter spaces with dependent parameters.

::::::::::::::::::::::::::::::::::::::::::::::::

By default, JUBE will split any parameter that contains the `,` (comma)
character into multiple tokens, using the comma character as the separator.
For each token, JUBE will create a separate workpackage.
This means a value of `hello,world`, will result in two distinct workpackages:
one with the corresponding parameter set to `hello`, the other one set to
`world`.

::: callout
JUBE allows for values containing the `,` (comma) character by specifying the separator manually for specific parameters.

:::::: group-tab

### XML

```xml
<parameter name="myparam" separator=";">1,2,3,4</parameter>
```

### YAML

```yaml
  parameter: {name: myparam, separator: ';', _:1,2,3,4}
```

::::::
:::

If multiple parameters can be tokenized, each parameter will become a dimenson
in the parameter space.
Consider the following example with two parameters containing comma-separated
values.

::: group-tab

### XML

```xml
<parameter name="processes">1,2</parameter>
<parameter name="threads">1,2,4</parameter>
```

### YAML

```yaml
  parameter: {name: processes, _: 1,2}
  parameter: {name: threads, _: 1,2,4}
```
:::

This will result in workpackages with the following parameter combinations:

|processes|threads|cores|
|--------:|------:|----:|
|1|1|1|
|1|2|2|
|1|4|4|
|2|1|2|
|2|2|4|
|2|4|8|

This make it easy to create larger parameter spaces along specific dimensions.
We can combine this to create a scaling experiment spanning multiple compute
nodes, with different process to thread ratios.

::::::::::::::::::::::::::::::::: challenge

Assume a node has 96 cores on two sockets (48 cores each). Create a configuration that runs a scalability study from 1 up to 4 nodes and tests different process to thread ratios but always uses all cores on the node. Apart from a configuration for a single process per node, the remaining configurations should have the same number of processes per socket (i.e., an even number of processes per node).

:::::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::: solution

With the constraints above, a configuration has

1. a single process, or
2. an even number of processes that is also a divisor of 96.

For this, we can set up multiple parameters that help us compute some of the parameters based on others.

::: group-tab

### XML

```xml
<parameter name="nodes">1,2,3,4</parameter>
<!-- Assuming a 2-socket node with 48 cores per socket -->
<parameter name="cores_per_node">96</parameter>
<parameter name="processes_per_node">1,2,4,8,12,24,48</parameter>
<parameter name="threads_per_process" mode="python">int(${cores_per_node}/${processes})</parameter>
```

### YAML

```yaml
  parameter:
  - { name: nodes, _: "1,2,3,4" }
  # Assuming a 2-socket node with 48 cores per socket -->
  - { name: cores_per_node, _: "96" }
  - { name: processes_per_node, _: "1,2,4,8,12,24,48" }
  - { name: threads_per_process, mode:python, _: "int(${cores_per_node}/${processes{})" }
```
:::

::::::::

Spanning parameter spaces can be used for different purposes.
As we see above, we can use this to create scalability studies where we scale the number of logical and/or physical computational entities of our HPC run.
However, this can be just as well used to span multiple different inputs or different combinations of arguments to the simulation.


::::::::::::::::::::::::::::::::::::: keypoints

- Comma-separated values in parameter definition are tokenized and create individual workpackages.
- Using multiple comma-separated values for parameters enables easy creation of
  parameter spaces.

::::::::::::::::::::::::::::::::::::::::::::::::


