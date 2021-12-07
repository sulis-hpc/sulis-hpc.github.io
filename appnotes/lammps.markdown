---
layout: page
title: LAMMPS
parent: Application Notes
nav_order: 3
---

# LAMMPS on Sulis
{: .no_toc }

1. TOC
{:toc}

LAMMPS contains a number of packages for tuning the performance and allowing for a different functionality. We have compiled the LAMMPS with the most important ones and performed some tests, which showed that `foss-2021b` toolchain gives the best performance in many cases, especially when running a CUDA-augmented build on a single GPU. Therefore, this one was used to build the LAMMPS on Sulis. Whether additional packages required, users are welcome to compile LAMMPS at the home directory using an example below.

## Accessing LAMMPS

First, check the list of existing LAMMPS modules

```bash
{{site.data.terminal.prompt}} module spider LAMMPS
```

Choosing a specific alias from the list e.g.,

```bash
{{site.data.terminal.prompt}} module spider LAMMPS/29Sep2021-kokkos-omp
```

will printout the required modules to be loaded prior to the LAMMPS' one, which are `GCC/11.2.0`, `OpenMPI/4.1.1` for the `foss-2021b` toolchain. More options are available if the code is built with multiple compiler toolchains. See the correspondence between compiler toolchain names and versions of included libraries at [this page](https://docs.easybuild.io/en/latest/Common-toolchains.html).

Finally, loading the `LAMMPS/29Sep2021-kokkos-omp` into the command line environment is invoked by

```bash
 module load GCC/11.2.0  OpenMPI/4.1.1 LAMMPS/29Sep2021-kokkos-omp
 ```

LAMMPS modules available so far (Dec 2021):

| module | description |
|-----------------------------------|----------------------------------|
| `LAMMPS/29Sep2021-kokkos-omp`			| built with OpenMP backend of `Kokkos` package |
| `LAMMPS/29Sep2021-CUDA-11.4.1-kokkos-omp`	| GPU-accelerated (via `Kokkos` package) build 	|

## Running on CPUs
There are many ways of running LAMMPS for which one may refer to the corresponding [documentation section](https://docs.lammps.org/Run_head.html) and which are not covered here. We list only a few examples of using several acceleration packages the LAMMPS installed with on the cluster.  

### Bare LAMMPS
The header of the submission script is similar for all CPU jobs.

<p class="codeblock-label">lammps.slurm</p>
```bash
#!/bin/bash
#SBATCH --nodes=1
#SBATCH --job=my_lammps_calculation
#SBATCH --ntasks-per-node=128
#SBATCH --cpus-per-task=1
#SBACTH --mem-per-cpu=3850
#SBATCH --time=24:00:00
#SBATCH --account=suxxx-somebudget

# drop all modules
module purge

# load ones required for the LAMMPS execuation
module load GCC/11.2.0 OpenMPI/4.1.1 LAMMPS/29Sep2021-kokkos-omp

# adjust the number of OpenMP threads automatically
export OMP_NUM_THREADS=${SLURM_CPUS_PER_TASK}

# executed command
srun `which lmp` -in in.lammps
```

Note, that `OMP_NUM_THREADS` is not used by the bare LAMMPS execution, which is not OpenMP-parallel, but it will be used below.

### OPT package
Shows good performance. Adding the `opt` suffix to the last line in the script above runs the [OPT package](https://docs.lammps.org/Speed_opt.html)

<p class="codeblock-label">lammps.slurm</p>
```bash
...
srun `which lmp` -sf opt -in in.lammps
```

### OMP package
The [OMP package](https://docs.lammps.org/Speed_omp.html) for the OpenMP-parallel calculations. The parallelisation is controlled by adjusting the `--ntasks-per-node` (number of MPI tasks) and `--cpus-per-task` (number of threads per MPI task) entries such that their product does not exceed the number of available cores on a node (128 for Sulis). For example, using 4 threads may be invoked by the following change to the script above

<p class="codeblock-label">lammps.slurm</p>
```bash
...
#SBATCH --ntasks-per-node=32
#SBATCH --cpus-per-task=4
...
srun `which lmp` --sf omp -in in.lammps
```

### Kokkos package
[Kokkos package](https://docs.lammps.org/Speed_kokkos.html) is built with `OpenMP` backed, and execution is controlled in the same fasion as of for the OMP package, with a slightly different `srun` command

<p class="codeblock-label">lammps.slurm</p>
```bash
...
srun `which lmp` -k on t ${OMP_NUM_THREADS} -sf kk -in in.lammps
```

The Kokkos may throw warnings stating that OMP_PROC_BIND and OMP_PLACES variables have to be set to "spread" and "threads" correspondingly (in addition to the OMP_NUM_THREADS above). One could play with that, although we did not notice a substantial effect on the performance.

## Running on GPUs
The [Kokkos](https://docs.lammps.org/Speed_kokkos.html) and [GPU](https://docs.lammps.org/Speed_gpu.html) packages allow for GPU-intensive calculations with LAMMPS. We see that the `Kokkos` is significantly faster on a single GPU. The `Kokkos` performance on two GPU cards is about the same the one for `GPU` package performance, and is significantly slower on three GPUs, This, however, may depend on a system of study and must be tested.

### Kokkos package
Available via `LAMMPS/29Sep2021-CUDA-11.4.1-kokkos-omp` module. Performs quite well as a single-tasks, single-GPU calculation with the following submission script

<p class="codeblock-label">lammps.slurm</p>
```bash
#!/bin/bash
#SBATCH --nodes=1
#SBATCH --job=my_lammps_calculation-gpu
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=42
#SBACTH --mem-per-cpu=3850
#SBATCH --time=24:00:00
#SBATCH --account=suxxx-somebudget
#SBATCH --partition=gpu
#SBATCH --gres=gpu:ampere_a100:1

# drop all modules
module purge

# load ones required for the LAMMPS execuation
module load GCC/11.2.0 OpenMPI/4.1.1 LAMMPS/29Sep2021-CUDA-11.4.1-kokkos-omp

# re-adjust the number of OpenMP threads
export OMP_NUM_THREADS=1

# executed command
srun `which lmp` -k on g 1 -sf kk -in in.lammps
```

It is important to note that `g 1` in the srun command line requests for a single GPU, which should be equal to the number of GPUs requested via `--gres` string (the number after last `:`).  The `--cpus-per-task=42` is used here dominantly for adjusting the memory request, since the `OMP_NUM_THREADS` is set to one. Note also that the number of requested GPUs must be equal to the number of MPI tasks. Last two conditions (`OMP_NUM_THREADS=1` and `SLURM_NTASKS_PER_NODE` equal to the number of GPU's per node) is chosen for the best performance, although users are encouraged to make their own tests in application to a particular situation.

### GPU package
The same conditions, `OMP_NUM_THREADS=1` and `SLURM_NTASKS_PER_NODE` equal to the number of GPU's per node, show the best performance. The corresponding submission script

<p class="codeblock-label">lammps.slurm</p>
```bash
#!/bin/bash
#SBATCH --nodes=1
#SBATCH --job=my_lammps_calculation-gpu
#SBATCH --ntasks-per-node=2
#SBATCH --cpus-per-task=42
#SBACTH --mem-per-cpu=3850
#SBATCH --time=24:00:00
#SBATCH --account=suxxx-somebudget
#SBATCH --partition=gpu
#SBATCH --gres=gpu:ampere_a100:2

# drop all modules
module purge

# load ones required for the LAMMPS execuation
module load GCC/11.2.0 OpenMPI/4.1.1 LAMMPS/29Sep2021-CUDA-11.4.1-gpu

# re-adjust the number of OpenMP threads
export OMP_NUM_THREADS=1

# executed command
srun `which lmp` -sf gpu -pk gpu 2 -in in.lammps
```

## Performance
We have tested foss-2020b, foss-2021b, iomkl-2019b and foss-2020a (for LAMMPS-3Mar2020 version) toolchains for CPU tasks, and derivatives (i.e., augmented with the cuda libraries) of foss-2020b (which is fosscuda-2020b) and foss-2021b for GPU runs. The testes system is an array of 55296 particles in the fcc geometry interacting via Lennard-Jones fields. All calculations are done on a single node, check [this page](https://github.com/arkdavy/LAMMPS) for more details. Tables below show time in seconds.

### CPU runs (MPI)

| toolchain | `foss-2020b` | `foss-2021b` | `iomkl-2019b` | `foss-2020a*` |
|:---:|:---:|:---:|:---:|:---:|
| bare LAMMPS | 99 | 98 | 108 | 102 |
| `OPT` package | 91 | 93 | 98 | 93 |


### CPU runs (MPI + OpenMP)

| toolchain |  number of threads  |
|:---:|:---:|:---:|:---:|:---:|
|           | 1 | 4 | 8 | 16 |
| `OMP` package |
|:---:|:---:|:---:|:---:|:---:|
| `foss-2020b` | 92 | 82 | 112 | 162 |
| `foss-2021b` | 92 | 84 | 113 | 163 |
| `iomkl-2019b` | 100 | 89 | 119 | 164 |
| `foss-2020a*` | 92 | 86 | 114 | 162 |
| `Kokkos` package |
|:---:|:---:|:---:|:---:|:---:|
| `foss-2020b` | 122 | 99 | 136 | 190 |
| `foss-2021b` | 100 | 92 | 129 | 176 |
| `iomkl-2019b` | 113 | 110 | 145 | 189 |
| `foss-2020a*` | 105 | 102 | 140 | 177 |

`*` LAMMPS version 3Mar2020

### GPU runs

| toolchain |  number of gpus  |
|:---:|:---:|:---:|:---:|:---:|
|           | 1 | 2 | 3 |
| Kokkos package |
|:---:|:---:|:---:|:---:|:---:|
| `foss-2020b` | 208 | 167 | 158 |
| `foss-2021b` | 183 | 146 | 148 |
| GPU package |
|:---:|:---:|:---:|:---:|:---:|
| `foss-2020b` | 268 | 143 | 120 |
| `foss-2021b` | 277 | 141 | 120 |



## Building LAMMPS
The following example illustrates the compilation of LAMMPS with the `GPU` package. Since the setup involves the GPU hardware detection, one has to make a compilation on the GPU partition by requesting an interactive session

```bash
salloc --account=suxxx-somebudget -p gpu -N 1 -n 1 -c 42 --mem-per-cpu=3850 --gres=gpu:ampere_a100:1 --time=8:00:00
```

Next, one should load the compiler toolchain which LAMMPS is intended to build with and the CUDA library

```bash
module load foss/2021b UCX-CUDA/1.11.2-CUDA-11.4.1 CMake/3.18.4
```

The `UCX-CUDA` loads the `CUDA/11.4.1` module and extends the OpenMPI to cuda-aware OpenMPI version allowing for a faster communication between MPI processes and GPUs. Such implementation is realised in `foss-2021a`, `foss-2021b` and will continue in the future. A separate toolchain was used in the past requiring a recompiled version of the OpenMPI library (e.g., fosscuda-2020b). For checking that `CUDA/11.4.1` module is loaded, enter `module list` command.

According to the [documentation](https://docs.lammps.org/Build.html) of LAMMPS, the recommended build option is with CMake. Inside the source directory

```bash
mkdir build
cd build
cmake -C ../cmake/presets/basic.cmake -D PKG_GPU=on -D GPU_API=cuda ../cmake

# to add another package, say BODY to the previous configuration you can run:
cmake -D PKG_BODY=on .
```

the build itself is done via

```bash
cmake --build .
```

And finally, modify the submission script for the `GPU` package calculations in its "module load" and "srun" parts

<p class="codeblock-label">lammps.slurm</p>
```bash
...
module load foss/2021b UCX-CUDA/1.11.2-CUDA-11.4.1
...
srun /path/to/lammps/source/build/lmp -sf gpu -pk gpu 2 -in in.lammps
```
