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

An out-of-the-box compilation of CASTEP 20.11 against the foss 2020b toolchain produces a build which passes all tests.

```terminal
{{site.data.terminal.prompt}} cd CASTEP-20.11
{{site.data.terminal.prompt}} module load foss/2020b
{{site.data.terminal.prompt}} export BUILD=fast
{{site.data.terminal.prompt}} export COMMS_ARCH=mpi
{{site.data.terminal.prompt}} export FFT=fftw3
{{site.data.terminal.prompt}} export MATHLIBS=openblas
{{site.data.terminal.prompt}} make
```

Paths to OpenBLAS and FFTW3 if not detected automatically can be interrogated via the environment variables `$EBROOTOPENBLAS` and `$EBROOTFFTW` after loading the foss/2020b module.

## Enhanced Performance with MKL 19

A significant performance uplift is available by using Intel MKL 19.5 in place of OpenBLAS. This requires setting the `MATHLIBS` variable above to `mkl` rather than `openblas` and specifying the path to Intel MKL as:

```terminal
/sulis/easybuild/software/imkl/2019.5.281-iimpi-2019b/compilers_and_libraries_2019.5.281/linux/mkl/lib/intel64_lin/
```

By default MKL will assume a minimal instruction set for non-Intel processors and give poor performance. It is therefore necessary to the set the following environment variables in SLURM job scripts which use MKL.

```bash
export MKL_DEBUG_CPU_TYPE=5
export MKL_CBWR=COMPATIBLE
```

In testing, use of MKL over OpenBLAS resulted in a 5-10% uplift in performance over OpenBLAS when running CASTEP benchmarks. Newer versions of MKL do not support this override. 

We intend to make the EasyBuild `gomkl` toolchain available in future to make this process simpler.

