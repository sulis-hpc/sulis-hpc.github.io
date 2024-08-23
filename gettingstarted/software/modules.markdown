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

{: .note }
The Sulis team make extensive use of [EasyBuild](https://easybuild.io) to compile and deploy software as environment modules. Users who wish to understand particular intricacies of the environment module setup on Sulis should consider consulting the [EasyBuild documentation](https://docs.easybuild.io/). 

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
{{site.data.terminal.prompt}} module load GCC/13.2.0
```
As a rule we do recommend you specify the module version anytime you load a module since the default version of a module will change over time as we deploy updated software.  

Typing module av with the GCC module loaded will yield additional modules that could now be loaded, such as OpenMPI (OpenMPI). In turn, loading OpenMPI will make the Scalable LAPACK (ScaLAPACK) module available for loading. 

## Searching modules

Manual module tree recursion isn't very practical to find software; `module spider` automates this process and should be used to search for software along with instructions on how to load specific modules. For example, to search for [OpenMM](https://openmm.org) try:

```plaintext
{{site.data.terminal.prompt}} module spider OpenMM
--------------------------
  OpenMM:
--------------------------
    Description:
      OpenMM is a toolkit for molecular simulation.
 
     Versions:
        OpenMM/8.1.2-CUDA-12.4.0
        OpenMM/8.1.2


This returned a list of all installed versions of OpenMM. To query the prerequisite modules that must first be loaded before attempting to load a particular version, query the specific version of interest.

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

    You will need to load all module(s) on any one of the lines below before the "OpenMM/{{site.data.software.ommver}}"
    module is available to load.

      {{site.data.software.defaultfoss}}
```

The modules listed as prerequisites correspond to a particular *toolchain*, i.e. a compiler (GCC) and MPI implementation (OpenMPI) against which this module has been built. Loading these two modules will also load modules that they, in turn, depend on.

In some cases we might be presented with multiple lists of prerequisites that we might load to enable `OpenMM`. This indicates that the software is available as separate builds against multiple toolchains. Load only **one** list of prerequisites, i.e. use a *single* toolchain.

Loading the prerequisites and the module itself will bring `OpenMM` into our environment. 

```shell
{{site.data.terminal.prompt}} module load {{site.data.software.defaultfoss}} OpenMM/{{site.data.software.ommver}}
```

{: .warning }
If loading multiple modules to create your software environment, always make sure these have a common set of non-conflicting requirements. For example a module which has prerequisites of `{{site.data.software.defaultfoss}}` can be loaded at the same time as one which only requires `{{site.data.software.defaultgcc}}`, but **not** at the same time as a module which requires `GCC/10.2.0`.

## GPU enabled modules

Some software has been built to take advantage of the GPU accelerators. In the case of `OpenMM` above this was listed as a separate module with the suffix `CUDA-{{site.data.slurm.defaultcuda}}` to indicate it depends on the Nvidia CUDA toolkit. Querying this module:

```shell
{{site.data.terminal.prompt}} module spider OpenMM/{{site.data.software.ommver}}-CUDA-{{site.data.software.defaultcudaver}}
```

This returns output similar to

```plaintext
--------------------------
  OpenMM: OpenMM/{{site.data.software.ommver}}-CUDA-{{site.data.software.defaultcudaver}}
--------------------------
    Description:
      OpenMM is a toolkit for molecular simulation.

    You will need to load all module(s) on any one of the lines below before the "OpenMM/{{site.data.software.ommver}}"
    module is available to load.

      {{site.data.software.defaultfoss}} {{site.data.software.defaultcuda}}
```

This indicates that we need to load `{{site.data.software.defaultcuda}}` in addition to `{{site.data.software.defaultfoss}}` before loading the build of `OpenMM` that supports CUDA.

{: .note }
Some older modules which use GPU acceleration are not labelled with a CUDA suffix but can be identified 
by the presence of CUDA in the list of prerequisite modules returned by `module spider`.



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






