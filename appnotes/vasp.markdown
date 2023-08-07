---
layout: page
title: VASP
parent: Application Notes
nav_order: 11
---

# VASP on Sulis
{: .no_toc }

1. TOC
{:toc}


The Vienna Ab initio Simulation Package ([VASP](https://www.vasp.at/info/about/)) is a computer program for atomic scale materials modelling, e.g. electronic structure calculations and quantum-mechanical molecular dynamics, from first principles. Due to the licensing limitations, we do not deploy it centrally, instead, a local institutional support team would be responsible for installations. Otherwise, follow the build instructions below tested for VASP-6.3.2. 

## VASP for CPU

### Building VASP
It is always advisable to build any code on a machine most similar to the one where it will run. Therefore, one should allocate an interactive session on a computational node before following the building instructions below

```bash
{{site.data.terminal.prompt}} salloc --account=suxxx-somebudget -N 1 -n 1 -c 4 --mem-per-cpu=3850 --time=8:00:00
```
After a limited benchmarking ([see below](#performance-data)), we have concluded that the optimal performance can be achived by compiling VASP with intel-2019b compiler toolchain and enabling the open OpenMP support. The corresponding compilers and libraries can be added to the environment via

```bash
[user@node042(sulis) ~]$ module load intel/2019b HDF5/1.10.5
```

The `makefile.include` below should be placed in the source code root
<details markdown="block" class="detail">
<summary>makefile.include</summary>
```bash
# Default precompiler options
CPP_OPTIONS = -DHOST=\"LinuxIFC\" \
              -DMPI -DMPI_BLOCK=8000 -Duse_collective \
              -DscaLAPACK \
              -DCACHE_SIZE=4000 \
              -Davoidalloc \
              -Dvasp6 \
              -Duse_bse_te \
              -Dtbdyn \
              -Dfock_dblbuf \
              -D_OPENMP

CPP         = fpp -f_com=no -free -w0  $*$(FUFFIX) $*$(SUFFIX) $(CPP_OPTIONS)

FC          = mpiifort -qopenmp
FCL         = mpiifort

FREE        = -free -names lowercase

FFLAGS      = -assume byterecl -w

OFLAG       = -O2
OFLAG_IN    = $(OFLAG)
DEBUG       = -O0

OBJECTS     = fftmpiw.o fftmpi_map.o fftw3d.o fft3dlib.o
OBJECTS_O1 += fftw3d.o fftmpi.o fftmpiw.o
OBJECTS_O2 += fft3dlib.o

# For what used to be vasp.5.lib
CPP_LIB     = $(CPP)
FC_LIB      = $(FC)
CC_LIB      = icc
CFLAGS_LIB  = -O
FFLAGS_LIB  = -O1
FREE_LIB    = $(FREE)

OBJECTS_LIB = linpack_double.o

# For the parser library
CXX_PARS    = icpc
LLIBS       = -lstdc++

##
## Customize as of this point! Of course you may change the preceding
## part of this file as well if you like, but it should rarely be
## necessary ...
##

# When compiling on the target machine itself, change this to the
# relevant target when cross-compiling for another architecture
VASP_TARGET_CPU ?= -xHOST
FFLAGS     += $(VASP_TARGET_CPU)

# Intel MKL (FFTW, BLAS, LAPACK, and scaLAPACK)
# (Note: for Intel Parallel Studio's MKL use -mkl instead of -qmkl)
FCL        += -mkl=parallel
MKLROOT    ?= /path/to/your/mkl/installation
LLIBS      += -L$(MKLROOT)/lib/intel64 -lmkl_scalapack_lp64 -lmkl_blacs_intelmpi_lp64 -liomp5 -lpthread -lm -ldl
INCS        =-I$(MKLROOT)/include/fftw

# HDF5-support (optional but strongly recommended)
CPP_OPTIONS+= -DVASP_HDF5
HDF5_ROOT  ?= /path/to/your/hdf5/installation
LLIBS      += -L$(HDF5_ROOT)/lib -lhdf5_fortran
INCS       += -I$(HDF5_ROOT)/include
```
</details>

Finally, working executables can be produced by the next build command

```bash
[user@node042(sulis) ~]$ make  -j 1 all HDF5_ROOT="$EBROOTHDF5" VASP_TARGET_CPU="-xCORE-AVX2"
```

### Running VASP
Using parallel OpenMP threads improves the VASP performance. In particular, two threads per MPI task appears to be optimal (subject to testing for particular system). It can be acheived by the following submission script

```bash
#!/bin/bash
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=64
#SBATCH --cpus-per-task=2
#SBATCH --mem-per-cpu=3850
#SBATCH --time=4:00:00
#SBATCH --account=suXXX-somebudget

module purge && module load intel/2019b HDF5/1.10.5

# check other OpenMP parameters here: https://www.vasp.at/wiki/index.php/Combining_MPI_and_OpenMP#For_the_OpenMP_runtime
export OMP_NUM_THREADS=${SLURM_CPUS_PER_TASK}
unset I_MPI_PMI_LIBRARY

vasproot=/location/of/the/VASP/installation

srun $vasproot/bin/vasp_std
```

## VASP for GPU

### Building VASP
An interactive session has to be allocated as for CPU build, but this time it must be on the GPU partition
```bash
{{site.data.terminal.prompt}} salloc --account=suxxx-somebudget -N 1 -n 1 -c 4 --mem-per-cpu=3850 --time=8:00:00 --partition=gpu --gres=gpu:ampere_a100:1
```

Then load the neccessary modules
```bash
[user@gpu042(sulis) ~]$ module purge
[user@gpu042(sulis) ~]$ module load GCC/11.2.0 FFTW/3.3.10
[user@gpu042(sulis) ~]$ module load NVHPC/21.11 OpenMPI/4.1.1-CUDAcore-11.5.1 ScaLAPACK/2.1.0
```

We have not yet produced a GPU build with recommended OpenMP support, therefore, the following makfile.include can be used for non-(CPU)threaded GPU build. Note that `FC` and `FCL` fortran compiler flags below contain site-specific (cc80) and environment-specific (cuda11.5) options. The first one corresponds to compute capability of Sulis' GPUs, while the second one corresponds to the CUDA library version loaded to the envirnment in the previos step

<details markdown="block" class="detail">
<summary>makefile.include</summary>
```bash
# Default precompiler options
CPP_OPTIONS = -DHOST=\"LinuxNV\" \
              -DMPI -DMPI_BLOCK=8000 -Duse_collective \
              -DscaLAPACK \
              -DCACHE_SIZE=4000 \
              -Davoidalloc \
              -Dvasp6 \
              -Duse_bse_te \
              -Dtbdyn \
              -Dqd_emulate \
              -Dfock_dblbuf \
              -D_OPENACC \
              -DUSENCCL -DUSENCCLP2P

CPP         = nvfortran -Mpreprocess -Mfree -Mextend -E $(CPP_OPTIONS) $*$(FUFFIX)  > $*$(SUFFIX)

# N.B.: you might need to change the cuda-version here
#       to one that comes with your NVIDIA-HPC SDK
FC          = mpif90 -acc -gpu=cc80,cuda11.5
FCL         = mpif90 -acc -gpu=cc80,cuda11.5 -c++libs

FREE        = -Mfree

FFLAGS      = -Mbackslash -Mlarge_arrays

OFLAG       = -fast

DEBUG       = -Mfree -O0 -traceback

OBJECTS     = fftmpiw.o fftmpi_map.o fftw3d.o fft3dlib.o

LLIBS       = -cudalib=cublas,cusolver,cufft,nccl -cuda

# Redefine the standard list of O1 and O2 objects
SOURCE_O1  := pade_fit.o
SOURCE_O2  := pead.o

# For what used to be vasp.5.lib
CPP_LIB     = $(CPP)
FC_LIB      = nvfortran
CC_LIB      = nvc -w
CFLAGS_LIB  = -O
FFLAGS_LIB  = -O1 -Mfixed
FREE_LIB    = $(FREE)

OBJECTS_LIB = linpack_double.o

# For the parser library
CXX_PARS    = nvc++ --no_warnings

# When compiling on the target machine itself , change this to the
# relevant target when cross-compiling for another architecture
VASP_TARGET_CPU ?= -tp host
FFLAGS     += $(VASP_TARGET_CPU)

# Improves performance when using NV HPC-SDK >=21.11 and CUDA >11.2
OFLAG_IN   = -fast -Mwarperf
SOURCE_IN  := nonlr.o

# Software emulation of quadruple precsion (mandatory)
QD         ?= $(NVROOT)/compilers/extras/qd
LLIBS      += -L$(QD)/lib -lqdmod -lqd
INCS       += -I$(QD)/include/qd

# BLAS (mandatory)
BLAS        = -lblas

# LAPACK (mandatory)
LAPACK      = -llapack

# scaLAPACK (mandatory)
SCALAPACK   = -L/lib -lscalapack

LLIBS      += $(SCALAPACK) $(LAPACK) $(BLAS)

# FFTW (mandatory)
FFTW_ROOT  ?= /path/to/your/fftw/installation
LLIBS      += -L$(FFTW_ROOT)/lib -lfftw3
INCS       += -I$(FFTW_ROOT)/include
```
</details>


Finally, build VASP with the following command
```bash
[user@gpu042(sulis) ~]$ make  -j 1 all FFTW_ROOT="$EBROOTFFTW" SCALAPACK_ROOT="$EBROOTSCALAPACK" NVROOT=$EBROOTNVHPC/Linux_x86_64/21.11
```
### Running VASP

A single-GPU job, which shows the best computational efficiency in our tests. It can be launched with the script below

```bash
#!/bin/bash
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=42
#SBATCH --mem-per-cpu=3850
#SBATCH --time=4:00:00
#SBATCH --account=suXXX-somebudget
#SBATCH --patition=gpu
#SBATCH --gres=gpu:ampere_a100:1

module purge
module load GCC/11.2.0 FFTW/3.3.10
module load NVHPC/21.11 OpenMPI/4.1.1-CUDAcore-11.5.1 ScaLAPACK/2.1.0

# Sometimes, 42 threads can be too many. Check if that many threads 
# indeed improve the perfomance over the number below
# Also, check other OpenMP parameters here: https://www.vasp.at/wiki/index.php/Combining_MPI_and_OpenMP#For_the_OpenMP_runtime
export OMP_NUM_THREADS=24
unset I_MPI_PMI_LIBRARY

vasproot=/location/of/the/VASP/installation

srun $vasproot/bin/vasp_std
```

## Performance data
Our choice for compiling and running instructions above is based on the one-node performance analys of different builds below. The tests we done for conjugate gradient relaxation on a system with 128 Lithium atoms per unit cell. We have only one data point for GPU perfromace at the time of writing (Q3 2023). We do not expect a significant effect of another GPU build on single-GPU calculations when using NVIDIA software, but improvement of multiple-GPU preformance due to different data transfer protocols is possible.


|toolchain|compiler|MPI implementation|MATHLIBS|time (s)| 
|---------|--------|------------------|--------|--------|
|           |                         |                |                  |omp1|omp2|omp4|omp8|
|           |                         |                |                  |----|---|---|---|
|foss/2021b |GCC/11.2.0               |OpenMPI/4.1.1   |OpenBLAS/0.3.18   |1364|   |   |   |
|           |                         |                |ScaLAPACK/2.1.0-fb|    |   |   |   |
|           |                         |                |FFTW/3.3.10       |    |   |   |   |
|foss/2021b |GCC/11.2.0               |OpenMPI/4.1.1   |AOCL/4.0          |1056|589|633|1869|
|intel/2019b|iccifort/2019.5.281      |impi/2019.7.217 |imkl/2019.5.281   |731 |381|414|830| 
|intel/2022a|intel-compilers/2022.1.0 |impi/2021.6.0   |imkl/2022.1.0     |917 |   |   |   | 
|           |                         |                |                  |gpu1|gpu2|gpu3|
|NVHPC/21.11|nvfortran                |OpenMPI/4.1.1   |OpenBLAS/0.3.20   |164|153|208| 
|           |                         |                |ScaLAPACK/2.1.0   |   |   |   | 
|           |                         |                |FFTW/3.3.10       |   |   |   | 

omp{\*}  and gpu{\*} labels differentiate the number of CPU threads and GPUs correspondingly.
