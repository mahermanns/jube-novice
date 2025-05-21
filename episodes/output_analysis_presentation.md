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

```output
gromacs_build:
|  compiler | compiler version |
|-----------|------------------|
| IntelLLVM |         2024.2.0 |

gromacs_run:
| wp | gromacs_core_time[s] | gromacs_wall_time[s] | gromacs_core_perf[ns/day] | gromacs_wall_perf[hours/ns] |
|----|----------------------|----------------------|---------------------------|-----------------------------|
|  2 |              150.776 |                6.282 |                   150.776 |                       6.282 |
```


::::::::::::::::::::::::::::::::::::: keypoints

- JUBE allows for definition of patterns to retrieve values from a workflow
  step.
- Patterns can be defined as regular expressions or Python expressions.
- JUBE allows for the generate of pretty-printed tables and CSV format.

::::::::::::::::::::::::::::::::::::::::::::::::



