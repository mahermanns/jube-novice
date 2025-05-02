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

::::::::::::::::::::::::::::::::::::: keypoints

- Use tags to include or exclude specific parts of a workflow.

::::::::::::::::::::::::::::::::::::::::::::::::

