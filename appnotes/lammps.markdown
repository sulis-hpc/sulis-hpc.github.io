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

LAMMPS contains a number of packages for tuning performance and for enabling additional functionality. We have compiled LAMMPS with the most commonly used packages and performed some basic tests for which the `foss-2021b` toolchain gave the best performance in many cases. This is especially true when running a CUDA-augmented build on a single GPU. The centrally provided LAMMPS module on Sulis uses this toolchain.

If additional packages are required then users are welcome to compile LAMMPS within their home directory following the [example below](#building-lammps).

## Accessing LAMMPS

First, check the list of existing LAMMPS modules

```bash
{{site.data.terminal.prompt}} module spider LAMMPS
```

Choosing a specific alias from the list e.g.,

```bash
{{site.data.terminal.prompt}} module spider LAMMPS/29Sep2021-kokkos-omp
```

will print out the required modules to be loaded before LAMMPS', which are `GCC/11.2.0`, `OpenMPI/4.1.1` for the `foss-2021b` toolchain. More options are available if the code is built with multiple compiler toolchains. See the correspondence between compiler toolchain names and versions of included libraries at [this page](https://docs.easybuild.io/en/latest/Common-toolchains.html).

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
There are many ways of running LAMMPS for which one may refer to the corresponding [documentation section](https://docs.lammps.org/Run_head.html) and which are not covered here. The next sections demonstrate the usage of available acceleration packages in LAMMPS.

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
srun lmp -in in.lammps
```

Note, that `OMP_NUM_THREADS` has no effect without the OMP package discussed below, but is included here for completeness.

### OPT package
This results in good performance in the provided builds provided suitable accelerated pair styles are available. Adding the `opt` suffix to the last line in the script above runs the [OPT package](https://docs.lammps.org/Speed_opt.html)

<p class="codeblock-label">lammps.slurm</p>
```bash
...
srun lmp -sf opt -in in.lammps
```

### OMP package
The [OMP package](https://docs.lammps.org/Speed_omp.html) using multiple threads per MPI task via OpenMP. The parallelisation is controlled by adjusting the `--ntasks-per-node` (number of MPI tasks) and `--cpus-per-task` (number of threads per MPI task) entries such that their product does not exceed the number of available cores on a node (128 for Sulis). For example, using 4 threads may be invoked by the following change to the script above

<p class="codeblock-label">lammps.slurm</p>
```bash
...
#SBATCH --ntasks-per-node=32
#SBATCH --cpus-per-task=4
...
srun lmp --sf omp -in in.lammps
```

We have noticed that some jobs with 8 per MPI task were unreasonably when using less than a full node. This was caused, most likely, by multiple OpenMP threads running on the same core. The issue was solved by using CPU binding:
<p class="codeblock-label">lammps.slurm</p>
```bash
...
srun --cpu-bind=cores lmp --sf omp -in in.lammps
```

### Kokkos package (CPU)
The [Kokkos package](https://docs.lammps.org/Speed_kokkos.html) is built with the `OpenMP` backend for use on the standard compute nodes. Execution is controlled in the same fashion as of for the OMP package with a slightly different `srun` command

<p class="codeblock-label">lammps.slurm</p>
```bash
...
srun lmp -k on t ${OMP_NUM_THREADS} -sf kk -in in.lammps
```

The Kokkos package may throw warnings stating that OMP_PROC_BIND and OMP_PLACES variables have to be set to "spread" and "threads" correspondingly (in addition to the OMP_NUM_THREADS above). Users may which to experiment with this. We have not noticed a substantial effect on the performance during our limited testing.

## Running on GPUs
The [Kokkos](https://docs.lammps.org/Speed_kokkos.html) and [GPU](https://docs.lammps.org/Speed_gpu.html) packages both provide GPU-acceleration for LAMMPS simulations which use compatible pair styles. In our testing the `Kokkos` package is resulted in significantly faster performance on a single GPU. The `Kokkos` performance on two GPU cards was similar to the `GPU` package performance and was significantly slower on three GPUs. The relative performance of the two packages is likely to be simulation-dependent and we encourage users to perform their own testing.

### Kokkos package
Available via `LAMMPS/29Sep2021-CUDA-11.4.1-kokkos-omp` module. The following submission script implements a single-task, single-GPU calculation.

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
srun lmp -k on g 1 -sf kk -in in.lammps
```

It is important to note that `g 1` in the srun command line requests a single GPU, which should be equal to the number of GPUs requested via `--gres` string (the number after last `:`).  The `--cpus-per-task=42` is used here dominantly for adjusting the memory request, since the `OMP_NUM_THREADS` is set to one. Note also that here the number of requested GPUs must be equal to the number of MPI tasks. Last two conditions (`OMP_NUM_THREADS=1` and `SLURM_NTASKS_PER_NODE` equal to the number of GPU's per node) is chosen for the best performance, although users are encouraged to make their own tests in application to a particular situation.

### GPU package
The corresponding submission script for the GPU package is below.

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
srun lmp -sf gpu -pk gpu 2 -in in.lammps
```

The same conditions as for Kokkos, `OMP_NUM_THREADS=1` and `SLURM_NTASKS_PER_NODE` equal to the number of GPU's per node, showed the best performance in our tests, however users may wish to experiment with multiple MPI tasks per GPU depending on their simulation details.

## Performance

Recommendations above are based on limited testing of a single system.

We have tested foss-2020b, foss-2021b, iomkl-2019b and foss-2020a (for LAMMPS-3Mar2020 version) toolchains for CPU tasks, and derivatives (i.e., augmented with the CUDA libraries) of foss-2020b (which is fosscuda-2020b) and foss-2021b for GPU runs. The tested system is an array of 55296 particles in the fcc geometry interacting via Lennard-Jones (LJ) fields. All calculations are done on a single node, check [this page](https://github.com/arkdavy/LAMMPS) for more details. The tables below show time in seconds.

The timings are obtained using all 128 CPUs of Sulis nodes, giving minimal interference between various calculations running on the cluster at the same time and making the tests results less biased. One should note, however, that the parallel efficiency (`T(1)/[N T(N)]` with `T(N)` being calculation time when using `N` processors) in this configuration is around ~50% for the chosen test case, which is quite low (see the plots [here](https://github.com/arkdavy/LAMMPS)). This may change when considering a different system, running other types of calculations or using more complicated forces. Our advise, however, is to use less cores in this situation (64 or less), targeting to the parallel efficiency of 75% or higher. We also encourage to estimate the parallel efficiency prior to full-scale runs and to find an optimal configuration in particular situation.

The best CPU performance was obtained with `OMP` package using two threads per MPI task. The `OPT` and `OMP` packages perform similarly well as single-threaded applications with 1 CPU per MPI task. Kokkos appears to be slightly slower, which is consistent with the [documentation](https://docs.lammps.org/Speed_kokkos.html).

The `foss-2021b` toolchain provides the quickest runs on a single GPU via the Kokkos package. As was mentioned earlier, one MPI rank on one CPU must be allocated per GPU. The `GPU` package gives a better scaling with GPUs count and may be considered as well.

### CPU runs (MPI)

| toolchain | `foss-2020b` | `foss-2021b` | `iomkl-2019b` | `foss-2020a*` |
|:---:|:---:|:---:|:---:|:---:|
| bare LAMMPS | 100 | 99 | 109 | 103 |
| OPT package | 89 | 90 | 97 | 93 |


### CPU runs (MPI + OpenMP)

| toolchain |  number of threads  |
|:---:|:---:|:---:|:---:|:---:|
|           | 1 | 2 | 4 | 8 |
| OMP package |
|:---:|:---:|:---:|:---:|:---:|
| `foss-2020b` | 91 | 79 | 95 | 132 |
| `foss-2021b` | 93 | 79 | 107 | 115 |
| `iomkl-2019b` | 98 | 87 | 89 | 119 |
| `foss-2020a` | 94 | 81 | 83 | 124 |
| Kokkos package |
|:---:|:---:|:---:|:---:|:---:|
| `foss-2020b` | 101 | 101 | 100 | 137 |
| `foss-2021b` | 100 | 89 | 92 | 126 |
| `iomkl-2019b` | 115 | 106 | 109 | 145 |
| `foss-2020a` | 105 | 102 | 102 | 140 |


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
The following instructions were tested on LAMMPS 29Sep2021 release available via the [link](https://www.lammps.org/download.html).

### CPU build
First, an interactive session must be requested on the CPU partition

```bash
salloc --account=suxxx-somebudget -N 1 -n 1 -c 42 --mem-per-cpu=3850 --time=8:00:00
```

Next, load the necessary modules sourcing the compiler toolchain, CMake and Python

```bash
module load foss/2021b CMake/3.21.1 Python/3.9.6
```
According to the [documentation](https://docs.lammps.org/Build.html) of LAMMPS, the recommended build option is with CMake. Inside the source directory,

```bash
mkdir build
cd build
cmake -C ../cmake/presets/most.cmake  -DBUILD_SHARED_LIBS=on ../cmake
```
The `most.cmake` selects many packages for installation. However, only packages with resolved dependencies will be configured. The CMake output normally informs which libraries are required to compile a package if it is not configured, and Sulis is likely to have those available via the module system. If a required package is not configured, loading missing modules and re-running the `cmake` command above would normally lead to successful configuration. The compilation itself is invoked via

```bash
cmake --build . -j 42
```

where the `-j` flag defines the parallelisation level of the build process and is taken as equal to the number of available cores requested in the interactive session.

To run the compiled code, modify the CPU submission script from e.g., `Bare LAMMPS` section in its "module load" and "srun" parts

<p class="codeblock-label">lammps.slurm</p>
```bash
...
module load foss/2021b Python/3.9.6
...
srun /path/to/lammps/source/build/lmp -in in.lammps
```

### GPU build
The next example illustrates the compilation of LAMMPS with the `GPU` package. Since the setup involves the GPU hardware detection, one has to make a compilation on the GPU partition by requesting an interactive session

```bash
salloc --account=suxxx-somebudget -p gpu -N 1 -n 1 -c 42 --mem-per-cpu=3850 --gres=gpu:ampere_a100:1 --time=8:00:00
```

Next, a compiler toolchain which LAMMPS is intended to build with must be loaded together with the CUDA library

```bash
module load foss/2021b UCX-CUDA/1.11.2-CUDA-11.4.1 CMake/3.21.1
```

The `UCX-CUDA` loads the `CUDA/11.4.1` module and extends the OpenMPI to cuda-aware OpenMPI version allowing for a faster communication between MPI processes and GPUs. Such implementation is realised in `foss-2021a`, `foss-2021b` and will continue in the future. A separate toolchain was used in the past requiring a recompiled version of the OpenMPI library (e.g., fosscuda-2020b). For checking that `CUDA/11.4.1` module is loaded, enter `module list` command. The configuration can be done with a basic LAMMPS preset, and GPU-related configuration flags.

```bash
mkdir build
cd build
cmake -C ../cmake/presets/basic.cmake -D PKG_GPU=on -D GPU_API=cuda -DGPU_ARCH=sm_80 -DBUILD_SHARED_LIBS=ON ../cmake
```

As with CPU build, the following compiles the code

```bash
cmake --build . -j 42
```

And finally, modify the submission script for the `GPU` package calculations above in its "module load" and "srun" parts

<p class="codeblock-label">lammps.slurm</p>
```bash
...
module load foss/2021b UCX-CUDA/1.11.2-CUDA-11.4.1
...
srun /path/to/lammps/source/build/lmp -sf gpu -pk gpu 2 -in in.lammps
```
