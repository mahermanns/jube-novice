---
title: "Wrap-up and outlook"
teaching: 10
exercises: 2
---


:::::::::::::::::::::::::::::::::::::: questions

- Which features of JUBE were not covered in this course?

::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: objectives

- Identify advanced features of JUBE.

::::::::::::::::::::::::::::::::::::::::::::::::

## Summary

In this course we created a minimal workflow to build and run the GROMACS application with a given input.
You can download the corresponding configurations [gromacs.xml](episodes/files/gromacs.xml) and [gromacs.yaml](episodes/files/gromacs.yaml) for further exploration.

## Outlook to other topics

In this workshop we have only scratched the surface of handling workflows with JUBE.
As with every software, inspiration for creative uses of specific features come with experience with those features.

Here is a list of some of the topics that could not be covered in this course

- Running multiple iterations of a step
- Re-evaluating parameter values at different
- More advanced combinations of parameter modes
- Advanced tagging with `duplicate` attribute for parameters
- Multiple `benchmark` definitions per file
- Shared steps
- Step cycles
- Using SQlite databases for results
- Debugging workflows


## Challenges with JUBE workflows

The main challenge with JUBE workflows is the separation of analysis and result tables from workflow steps.
This makes it hard to define additional steps that analyse output and create result tables.

The easiest way to cope with this is to create additional scripts outside of the workflow that act on the created tables (e.g. creating plots, etc.).
However, this is unsatisfying as the means part of our overall workflow (i.e., the generation of plots) is outside of the actual workflow specification and has to be done manually.

Another option is to have a second layer of workflow specifications where the execution of a JUBE workflow is a step in a parent workflow, also defined in JUBE.
This way, the full workflow can be defined in JUBE, yet the additional layer add complexity overall.

Finally, as analyses can be performed and result tables created and updated while the workflow is not yet finished, another viable approach can be to add a `do` action with a `done_file` at the location where the workflow should pause until the result tables are complete, and then create the `done_file` manually once all workpackages have completed up to that point.

::::::::::::::::::::::::::::::::::::: keypoints

- JUBE is a very flexible workflow management system.
- JUBE's advanced features can be further customized with Python and Perl expressions.

::::::::::::::::::::::::::::::::::::::::::::::::




