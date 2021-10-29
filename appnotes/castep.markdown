---
layout: page
title: CASTEP
parent: Application Notes
nav_order: 3
---

# CASTEP on Sulis
{: .no_toc }

1. TOC
{:toc}

These notes are a work in progress and do not represent a final position on how to get best performance from CASTEP on Sulis.

## Compilation 

An out-of-the-box compilation of CASTEP 20.11 against the gomkl/2019b toolchain produces a build which passes all tests. Performance
is significantly better (10-15%) using MKL 19 via this toolchain rather than OpenBLAS via the foss toolchains, but only if setting

```bash
export MKL_DEBUG_CPU_TYPE=5
export MKL_CBWR=COMPATIBLE
```

to force detection of a suitable instruction set for the AMD processors in Sulis. These variables are set automatically when loading
the gomkl toolchain.

To build CASTEP from source:

```terminal
{{site.data.terminal.prompt}} cd CASTEP-20.11
{{site.data.terminal.prompt}} module load gomkl/2019b FFTW/3.3.8
{{site.data.terminal.prompt}} export BUILD=fast
{{site.data.terminal.prompt}} export COMMS_ARCH=mpi
{{site.data.terminal.prompt}} export FFT=fftw3
{{site.data.terminal.prompt}} export FFTLIBDIR=$EBROOTFFTW
{{site.data.terminal.prompt}} export MATHLIBS=mkl
{{site.data.terminal.prompt}} export MATHLIBDIR=$MKLROOT/lib/intel64_lin/
{{site.data.terminal.prompt}} make
```

Job scripts which use CASTEP built in this way should load the gomkl and FFTW modules before invoking the `castep.mpi` executable.

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
module load gomkl/2019b FFTW/3.3.8

srun --cpu-bind=rank castep.mpi
```

