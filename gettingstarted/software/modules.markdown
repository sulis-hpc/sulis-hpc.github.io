---
layout: page
title: Environment modules
parent: Software
grand_parent: Getting Started
nav_order: 1
---

# Environment modules
{: .no_toc }

1. TOC
{:toc}


## About environment modules

Like many HPC platforms, Sulis uses an environment module system as a means for users to import the required software into their execution environment. Environment modules provide a convenient means to customise  software environments at the user level, i.e. without making system level changes which may break compatibility with software other users depend upon.

Other tools, such as [Anaconda](https://www.anaconda.com/), allow similar customisation. However software provided via the environment module system has been specifically built and optimised for the Sulis hardware. Use of environment modules should therefore be the default choice for most users to access software. Please search the available software (see below) before requesting new software be installed.

As well as specific scientific applications, environment modules provide access to software development *toolchains*, i.e. combinations of compilers, communication libraries (i.e. MPI) and optimised numerical libraries (BLAS, LaPACK, FFTW etc) that have been tested and known to work well on the Sulis hardware. See the section on [compiling](compiling) for more information.

Sulis provides a set of software available to all users via environment modules available at login. Users at specific universities/institutions may be given access to additional modules created by their local research computing support team, for example if commercial software is only licensed for use at a particular site.

## Accessing modules

Modules are presented as a hierarchy such that initially only a set of top level core modules are available. Once a core module is loaded, additional dependent, compatible modules will become available and so on. To view available modules, use: 

```shell
{{site.data.terminal.prompt}} module av
```

If no modules are loaded, only core modules will be displayed. To load a module, for example:

```shell
{{site.data.terminal.prompt}} module load GCC
```
which will load the *default* version of the GNU GCC compiler. To load a specific version of the GCC compiler: 

```shell
{{site.data.terminal.prompt}} module load GCC/10.2.0
```
As a rule we do recommend you specify the module version anytime you load a module since (on rare occasions) we may find it necessary to change the default version of a module. 

Typing module av with the GCC module loaded will yield additional modules that could now be loaded, such as OpenMPI (OpenMPI). In turn, loading OpenMPI will make the Scalable LAPACK (ScaLAPACK) module available for loading. 

## Searching modules

Manual module tree recursion isn't very practical to find software; `module spider` automates this process and should be used to search for software along with instructions on how to load specific modules. For example, to search for [OpenMM](https://openmm.org) try:

```shell
{{site.data.terminal.prompt}} module spider OpenMM
```

This returns a list of all installed versions of OpenMM. To query the pre-requisite module that must first be loaded before attempting to load a particular version, query the specific version of interest.

```shell
{{site.data.terminal.prompt}} module spider OpenMM/{{site.data.software.ommver}}
```

This returns output similar to

```plaintext
--------------------------
  OpenMM: OpenMM/{{site.data.software.ommver}}
--------------------------
    Description:
      OpenMM is a toolkit for molecular simulation.

    You will need to load all module(s) on any one of the lines below before the "OpenMM/7.5.1"
    module is available to load.

      {{site.data.software.defaultfosscuda}}
      {{site.data.software.defaultfoss}}
```

In this example we are presented with **two** sets of modules that we might load as prerequisites for loading the OpenMM module. We must only load **one** of them. The first includes CUDA and should only be used if making use of GPU acceleration - see [GPU jobs](../batchq/gpu/). The second should be loaded for use on the standard compute nodes, i.e.

```shell
{{site.data.terminal.prompt}} module load  {{site.data.software.defaultfoss}} OpenMM/{{site.data.software.ommver}}
```

## Other module commands

Currently loaded environment modules can be listed with,

```shell
{{site.data.terminal.prompt}} module list
```

and specific modules can be unloaded using, for example:

```shell
{{site.data.terminal.prompt}} module unload OpenMM
```

To unload all loaded modules use:
```shell
{{site.data.terminal.prompt}} module purge
```

It is strongly recommended that scripts submitted to [batch queues](../../batchq/) invoke `module purge` before loading any modules to create a software environment for the job.






