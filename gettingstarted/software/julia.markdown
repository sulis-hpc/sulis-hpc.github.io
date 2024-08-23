---
layout: page
title: Julia 
parent: Software
grand_parent: Getting Started
nav_order: 4
---

# Julia
{: .no_toc }

The [Julia language](https://julialang.org) aims to provide a compromise between a high level interpreted language like Python and a high performance compiled language like C or Fortran. It is explicitly designed for numerical computing. 

1. TOC
{:toc}

## Accessing Julia

Julia can be downloaded and installed into your user home directory on Sulis. No administrative permissions are required to run the installer.

- [Download page at julialang.org](https://julialang.org/downloads/)

We also provide Julia centrally via the [environment module](modules.markdown) system. 



```shell
{{site.data.terminal.prompt}} module spider julia
    ----------------------------------------------------------------------------
      Julia: Julia/{{site.data.software.juliamodver}}
    ----------------------------------------------------------------------------
        Description:
        Julia is a high-level, high-performance dynamic programming language for 
        numerical computing

        This module can be loaded directly: module load Julia/{{site.data.software.juliamodver}}
```

Following the advice printed by that ``module spider`` output we can load the module and enter the Julia interpreter.

```shell
{{site.data.terminal.prompt}} module load Julia/{{site.data.software.juliamodver}}
{{site.data.terminal.prompt}} julia
                   _
       _       _ _(_)_     |  Documentation: https://docs.julialang.org
      (_)     | (_) (_)    |
       _ _   _| |_  __ _   |  Type "?" for help, "]?" for Pkg help.
      | | | | | | |/ _` |  |
      | | |_| | | | (_| |  |  Version {{site.data.software.juliamodver}}
     _/ |\__'_|_|_|\__'_|  |  Official https://julialang.org/ release
    |__/                   |

    julia> 
```

If you need a newer version of Julia than that installed centrally, feel free to request it be updated by contacting [support](../../support.markdown).

## Running Julia scripts

Scripts written in the Julia language are usually given the ``.jl`` extension and can be launched at the command line or within [SLURM job scripts](../batchq/index.markdown) scripts by providing the name of the script as an argument to the ``julia`` command as follows.

```shell
{{site.data.terminal.prompt}} julia my_script.jl
```

In the above it is assumed that the Julia script ``my_script.jl`` exists in the working directory where ``julia`` is invoked.


## Installing Julia packages

Users may need to install additional Julia packages. By default Julia tries to install these into the same 
location where Julia itself is installed. If using the installation of Julia provided centrally via the [environment module](modules.markdown) system this will fail. Instead users need to set the ``JULIA_DEPOT_PATH`` environment variable` to a location in their home directory. Julia will then install new packages to that location. For example:

```shell
{{site.data.terminal.prompt}} export JULIA_DEPOT_PATH=~/julia
```

Users may wish to set this environment variable for use in all future sessions by modifying the appropriate login file (`.bashrc`).

With this variable set, installation of packages should be straightforward. For example to install the `CSV` package within the Julia interpreter:

```julia
    julia> import Pkg
    julia> Pkg.add("CSV")
```

{: .warning }
The behaviour of Julia regarding where packages are installed by default does seem to change frequently and often doesn't match that described in the official documentation. If in doubt contact [support](../../support.markdown). 

## Julia interactive notebooks

Julia can be used as a kernel with the [Jupyter](https://jupyter.org/) system by installing the `IJulia` package. This will pull in a great number of dependencies. On Sulis systems we recommended only installing and using the IJulia package after loading `JupyterNotebook` and other environment modules which will already provide most of these dependencies.

```shell
{{site.data.terminal.prompt}} module purge
{{site.data.terminal.prompt}} module load {{site.data.software.defaultgcc}}
{{site.data.terminal.prompt}} module load JupyterNotebook/{{site.data.software.defaultgcc}}
{{site.data.terminal.prompt}} module load {{site.data.software.defaultscipy}} {{site.data.software.mplmodule}}
{{site.data.terminal.prompt}} module load Julia/{{site.data.software.juliamodver}}
{{site.data.terminal.prompt}} julia -e 'import Pkg;Pkg.add("IJulia")'
```

Notebook servers launched using the command ``jupyter notebook`` will then have the Julia kernel available. 

{: .important}
Jupyter notebook servers should not be launched on the login node, but from within an [interactive job](../batchq/interactive.markdown) on a compute node.

Please study the [application note on Jupyter](../../appnotes/jupyter.markdown) for information on how to connect to the remote notebook.

## Parallelism in Julia

Two popular ways of exploiting parallelism within a single Julia script are documented at the relevant pages linked below.

- Using the [distributed package](../batchq/singlenode.markdown#julia-distributed).
- Using the [MPI.jl package](../batchq/mpi.markdown#using-mpijl-in-julia).