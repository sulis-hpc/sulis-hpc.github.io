---
layout: page
title: Compiling code 
parent: Software
grand_parent: Getting Started
nav_order: 2
---

# Compiling code
{: .no_toc }

1. TOC
{:toc}

## Before you start

Before compiling your own code on Sulis, please [search the software already available](modules/#searching-modules) via the module system.

If the software you intend to build is likely to be needed by multiple users at your site or elsewhere then consider requesting a centrally installed build via your local [research computing support team](../../support)(HPC Midlands+ consortium members) or by raising an issue in the [sulis-hpc/support](https://github.com/sulis-hpc/support/issues) repository on GitHub (EPSRC national access users).

If creating bespoke/modified builds of commonly used software, please check the [Application Notes](../../appnotes) section. This may contain advice on build settings for getting optimal performance from the Sulis hardware. If no such note is available consider working with your local Research Software Engineering (RSE) team to contribute a new application note.

The environment module system can be used to access compilers and optimised libraries useful for creating your own builds of compiled software. These are organised into *toolchains*. Most software on Sulis has been built using the [EasyBuild](https://easybuild.io/) software build and installation framework, and hence the available toolchains are closely aligned to those used within that framework.  A toolchain consists of at least a compiler suite, and then optionally an MPI implementation, optimised BLAS/LaPACK libraries, the ScaLAPACK parallel library, and the FFTW Fourier Transform library. See the [common toolchains](https://docs.easybuild.io/en/latest/Common-toolchains.html) (EasyBuild project) page for more information. The compilation instructions below use elements of these toolchains.

A very common pitfall when working with HPC systems is to mismatch modules used at this stage (i.e. compile time) and modules loaded in batch scripts which use the resulting software (i.e. run time). Do please take notes of your build environment, and be aware that automated build systems such as GNU Make will not automatically rebuild software from scratch if you unload one set of compilers/libraries and load another. Consider starting a new build in a seperate directory or using `make clean` (if the software has a makefile) if switching to a different toolchain.

## Serial code

It may be necessary to compile single-threaded/serial applications for subsequent use in an ensemble computing workflow. 

For most codes, the latest version of the GCC compiler suite should be the most appropriate. Note that this includes compilers for Fortran as well as C/C++ code. 

1. Load the compiler module.
   ```shell
   {{site.data.terminal.prompt}} module load {{site.data.software.defaultgcc}}
   ```
2. Invoke the appropriate compiler in the usual way.

    1. For Fortran code:
       ```shell
       {{site.data.terminal.prompt}} gfortran myserialcode.f90
       ```
    2. For C code:
       ```shell
       {{site.data.terminal.prompt}} gcc myserialcode.c
       ```
    3. For C++ code:
       ```shell
       {{site.data.terminal.prompt}} g++ myserialcode.cpp
       ```

More substantial codes might be provided with a makefile, in which case the appropriate compiler should be specified when modifying this for Sulis. Other codes generate a Makefile automatically via a configure script or via CMake.  When compiling these codes be sure to load the appropriate compiler/library modules *before* generating the makefile. 

## OpenMP code

Many scientific codes that use threaded parallelism to exploit multiple processor cores within a single compute node are written using OpenMP. Such codes may also form part of an ensemble computing workflow that launches many copies of a code accross multiple nodes, with each copy making use of all, or part of, a single node. 

The appropriate OpenMP flag should be added when compiling.

1. Load the compiler module.
   ```shell
   {{site.data.terminal.prompt}} module load {{site.data.software.defaultgcc}}
   ```
2. Invoke the appropriate compiler in the usual way.

    1. For Fortran code:
       ```shell
       {{site.data.terminal.prompt}} gfortran -fopenmp myopenmpcode.f90
       ```
    2. For C code:
       ```shell
       {{site.data.terminal.prompt}} gcc -fopenmp myopenmpcode.c
       ```
    3. For C++ code:
       ```shell
       {{site.data.terminal.prompt}} g++ -fopenmp myopenmpcode.cpp
       ```

The number of theads launched by each instance of the code at runtime is usually controlled by the `OMP_NUM_THREADS` environment variable. More information is given in the section on [batch queues](../batchq/). 

## Message Passing (MPI) code

Software written to use multiple nodes in an HPC cluster often uses MPI to implement communucation between multiple processes. Compiling MPI code requires loading an appropriate MPI module and compiling using the appropriate compiler wrapper rather than invoking the underlying compilers directly.

1. Load compiler and MPI modules from a toolchain. 
   ```shell
   {{site.data.terminal.prompt}} module load {{site.data.software.defaultfoss}}
   ```
2. Invoke the appropriate compiler wrapper.

    1. For Fortran code:
       ```shell
       {{site.data.terminal.prompt}} mpif90 mympicode.f90
       ```
    2. For C code:
       ```shell
       {{site.data.terminal.prompt}} mpicc mympicode.c
       ```
    3. For C++ code:
       ```shell
       {{site.data.terminal.prompt}} mpicxx mympicode.cpp
       ```

As with serial and OpenMP code, it may be necessary to specify these wrapper commands in an appropriate makefile, or to load the modules at step 1 before using a configure script or cmake to generate one.

Some codes use both OpenMP and MPI in a hybrid fashion, i.e. with each process using multiple threads. For such codes the `-fopenmp` flag should be added as an argument to the compiler wrapper.

## Compiling for GPUs

Code written using CUDA C to take advantage of the Sulis A100 GPUs should be compiled using `nvcc` after loading the appropriate CUDA toolkit module, and should target compute capability 8.0. Code written to use the L40 GPUs should target compute capability 8.9. Both can be specified simultaneously as below.

1. Load compiler and CUDA modules from a toolchain. 
   ```shell
   {{site.data.terminal.prompt}} module load {{site.data.software.defaultgcc}} {{site.data.software.defaultcuda}}
   ```
2. Invoke the CUDA C compiler.
   ```shell
   {{site.data.terminal.prompt}} nvcc --gpu-code=sm_80 --gpu-code=sm_89 mycudacode.cu 
   ```
Note that CUDA toolkit version 11.8 or higher is needed to target compute capability 8.9.


CUDA-aware MPI modules are also available and can be identified via `module spider`. For example:
```shell
{{site.data.terminal.prompt}} module spider {{site.data.software.defaultmpi}}
```
The prerequisite list which includes a CUDA module should be loaded for access to CUDA-aware MPI compiler wrappers. For example:
```shell
{{site.data.terminal.prompt}} module load {{site.data.software.defaultfosscuda}}
```

WARNING. Software compiled to use use CUDA will only execute on the GPU nodes accessible via the {{site.data.slurm.gpunode_partition_name}} queue. It will not run on the login node or the standard compute nodes and will likely produce errors of the form:
```plaintext
libcuda.so.1: cannot open shared object file: No such file or directory.
```
This is expected. Only the GPU nodes have access to the CUDA runtime library.

<!--- Support for CUDA Fortran is available via the Nvidia HPC SDK ??? --->

## Linking libraries

Many software build systems will correctly detect and link against libraries which have been loaded via [environment modules](modules). If linking against libraries manually, the `pkg-config` tool may be useful.

As an example, to compile code which uses the [FFTW3](http://fftw.org/) library we would first load an appropriate module.
```shell
{{site.data.terminal.prompt}} module load {{site.data.software.defaultfoss}} FFTW/3.3.8
```
After which 
```shell
{{site.data.terminal.prompt}} pkg-config --cflags fftw3
```
will return the flags needed to instruct the compiler where to search for the fftw3.h header file (C/C++) or Fortran include files needed to use that library. Similarly
```shell
{{site.data.terminal.prompt}} pkg-config --libs fftw3
```
will return the flags needed to link your code against the library.

These commands can be used with backtick substitution as (for example)
```shell
{{site.data.terminal.prompt}} gcc `pkg-config --cflags fftw3` my_code.c `pkg-config --libs fftw3` 
```
