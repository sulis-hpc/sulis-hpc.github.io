---
layout: page
title: CASTEP
parent: Application Notes
nav_order: 1
---

# CASTEP on Sulis
{: .no_toc }

1. TOC
{:toc}

These notes are a work in progress and do not represent a final position on how to get best performance from CASTEP on Sulis. We anticipate that most use of CASTEP on Sulis will be via workflows that involve many calculations using a single node (or less). We have not studied
multi-node performance in any detail.

## Compilation 

An out-of-the-box compilation of CASTEP 20.11 against the foss/2020b toolchain will pass all build tests. This has produced the best performance in our testing (see below).

To build CASTEP from source:

```terminal
{{site.data.terminal.prompt}} cd CASTEP-20.11
{{site.data.terminal.prompt}} module load foss/2020b
{{site.data.terminal.prompt}} export BUILD=fast
{{site.data.terminal.prompt}} export COMMS_ARCH=mpi
{{site.data.terminal.prompt}} export FFT=fftw3
{{site.data.terminal.prompt}} export FFTLIBDIR=$EBROOTFFTW
{{site.data.terminal.prompt}} export MATHLIBS=openblas
{{site.data.terminal.prompt}} export MATHLIBDIR=$EBROOTOPENBLAS
{{site.data.terminal.prompt}} make
```

Job scripts which use CASTEP built in this way should load the foss/2020b module before invoking the `castep.mpi` executable.

## Running

CASTEP benefits from pinning of MPI tasks to CPU cores by rank. We recommend launching CASTEP as per the following example job script.

<p class="codeblock-label">castep.slurm</p>
```bash
#!/bin/bash
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=128
#SBATCH --mem-per-cpu=3850
#SBATCH --time=08:00:00
#SBATCH --account=suxxx-somebudget

module purge
module load foss/2020b

srun --cpu-bind=rank castep.mpi <seedname>
```
Where `<seedname>` is the usual argument to CASTEP.

## Performance

Performance testing has been limited to the [Al3x3](http://www.castep.org/CASTEP/Benchmarks) (medium) benchmark test. Results when compiled with various compiler/library combinations are below. All tests use FFTW 3.3.8  for 
Fourier transforms.

| toolchain   |   compiler                | MPI implementation | MATHLIBS             |  time (s)  | 
|-------------|---------------------------|--------------------|----------------------|------------|
| foss/2020b  | GCC/10.2.0                |   OpenMPI/4.0.5    | OpenBLAS/0.3.12      |  305.8(1)  |
| gomkl/2019b | GCC/8.3.0                 |   OpenMPI/3.1.3    | imkl/2019.5.281 <sup id="a1">[1](#f1)</sup> |  310(2)    |
| intel/2019b | iccifort/2019.5.281  <sup id="a2">[2](#f2)</sup> |   impi/2019.7.217  | imkl/2019.5.281 <sup id="a1">[1](#f1)</sup> |  324.5(5)  | 
| iomkl/2019b | iccifort/2019.5.281  <sup id="a2">[2](#f2)</sup> |   OpenMPI/3.1.4    | imkl/2019.5.281 <sup id="a1">[1](#f1)</sup> |  334.5(4)  | 

 1. <small id="f1"> Uses environment variables `export MKL_DEBUG_CPU_TYPE=5 ; export MKL_CBWR=COMPATIBLE` such that
MKL uses appropriate instruction sets for AMD EPYC.
  </small> [↩](#a1)
2. <small id="f2"> Makefile edited to force `OPT_CPU =  -march=core-avx2` for use of appropriate instruction set.
  </small> [↩](#a2)


Time reported is the "Calculation time" output by the CASTEP internal timer.

Note that Sulis uses turboboost such that performance of the AMD EPYC processors is increased beyond the default clock speed if there is sufficient power and thermal headroom. Performance can vary between nodes and times. All the above measurements were taken on the same node within a few minutes of each other.

At the time of writing, performance when using [AOCL](https://developer.amd.com/amd-aocl/) for BLAS/LAPACK was significantly worse than anything in the above table. 

