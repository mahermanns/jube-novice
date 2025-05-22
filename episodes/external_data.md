---
title: "Including external data"
teaching: 10
exercises: 5
---

:::::::::::::::::::::::::::::::::::::: questions

- How to better structure your workflow files to allow for reuse?
- How to reduce redundancy of definitions?

::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: objectives

- List different mechanisms to include data from multiple configuration files.
- Select an appropriate inclusion mechanism in different scenarios.

::::::::::::::::::::::::::::::::::::::::::::::::

When workflows become complex it often helps to organize parts of those workflows in different files.
JUBE provides three different ways to include configuration data from different files.

## Reusing external sets

The simplest form of reusing external configuration data is to define a set in an external file and use it as is in a different configuration file as part of a step definition.

:::::::::::::::::::: group-tab
### XML
```sh
$ nano external_config.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<jube>

  <parameterset name="numbers_pset">
    <parameter name="number" type="int">1,2,4</parameter>
  </parameterset>

  <parameterset name="strings_pset">
    <parameter name="text">Hello</parameter>
  </parameterset>
</jube>
```
```sh
$ nano main.xml
```
```xml
<step name="mystep">
  <use from="external_config.xml">numbers_pset</use>
</step>
```
### YAML
```sh
$ nano external_config.yaml
```yaml
parameterset:
  - name: numbers_pset
    parameter:
    - { name: number, type: int, _: "1,2,3,4" }
  - name: strings_pset
    parameter:
    - { name: text, _: Hello World }
```
```sh
$ nano main.yaml
```
```yaml
step:
  - name: mystep
    use:
      - { from: external_config.yaml, _: numbers_pset }
```
::::::::::::::::::::::::::::::

## Initializing local sets with external data

Sometimes values defined in external sets work almost for a different workflow scenario, yet small changes would be necessary.
Copying the set and modifying the small number of values to be changed would create a lot of redundancy.
For such situations JUBE allows users to define sets and initialize them with the values from a different set.
The new set contains all values of the external set, and only needs to redefine the values that need changing.

:::::::::::::: callout
Next to tagging, this is another way to change the value of a configuration entity in JUBE.
::::::::::::::::::::::

:::::::::::::::::::: group-tab
### XML
```sh
$ nano main.xml
```
```xml
<parameterset name="numbers_pset" init_with="external_config.xml:numbers.xml">
  <parameter name="number" type="int">1,2</parameter>
</parameterset>
<step name="mystep">
  <use>numbers_pset</use>
</step>
```
### YAML
```sh
$ nano main.yaml
```
```yaml
parameterset:
  - name: numbers_pset
    init_with: "external_config.yaml:numbers_pset"
    parameter:
    - { name: number, type: int, _: "1,2" }
step:
  - name: mystep
    use: numbers_pset
```
::::::::::::::::::::::::::::::

## Including arbitrary configuration data

So far, we only reused set defined in external files.
Howerver, JUBE also allows to include arbitrary parts of an external configuration file.
The external configuration file is even allowed to contain non-JUBE tags (e.g., to make it easier to identify a block of entities), as long as the included portion of the external configuration does not retain any unknown tags.

The specifics of this is out of the scope for this tutorial at this stage, so please reffer to the
[JUBE tutorial pages of the Jülich Supercomputing Centre][jube-tutorial] for more details on that.

:::::::::::::::::::::::::::::::::::::: keypoints

- `from` inside a `use` clause allows to include an external set as is.
- `init_with` allows to initialize child sets with the values of a parent set and modify and extend the set.
- `include` allows to include arbitrary parts of external configuration files.

::::::::::::::::::::::::::::::::::::::::::::::::