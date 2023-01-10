---
layout: page
title: QuantumESPRESSO
parent: Application Notes
nav_order: 8
---

# QuantumESPRESSO on Sulis
{: .no_toc }

1. TOC
{:toc}

[QuantumESPRESSO](https://www.quantum-espresso.org) is a suite of codes for ab-initio modelling based on plane waves and pseudopotentials with the Density Functional Theory (DFT) in its core.

## Accessing QuantumESPRESSO

First, check the list of existing `QuantumESPRESSO` modules

```bash
{{site.data.terminal.prompt}} module spider QuantumESPRESSO
```

Choosing a specific alias from the list e.g.,

```bash
{{site.data.terminal.prompt}} module spider QuantumESPRESSO/7.1
```

will print out the required modules to be loaded before `QuantumESPRESSO` one.

Loading the `QuantumESPRESSO/7.1` into the command line environment is invoked by

```bash
module load GCC/11.3.0  OpenMPI/4.1.4
module load QuantumESPRESSO/7.1
 ```

## Running QuantumESPRESSO
Version `7.1` and onwards, set the parallel configuration automatically. We, therefore, can safely recommend running `QuantumESPRESSO/7.1` without any parallel options.

```bash
#SBATCH --nodes=4
#SBATCH --ntasks-per-node=128
#SBATCH --cpus-per-task=1
#SBATCH --mem-per-cpu=3850
#SBATCH --time=15:00:00
#SBATCH --account=suXXX-somebudget

module purge
module load GCC/11.3.0 OpenMPI/4.1.4
module load QuantumESPRESSO/7.1

srun pw.x -i my.input
```

With earlier vesion, e.g. `6.8`, the parallelisation of QuantumESPRESSO can be tuned manually by distributing processors across groups defined by the command line switches: -nimage, -npools, -nband, -ntg, -ndiag or -northo (shorthands, respectively: -ni, -nk, -nb, -nt, -nd).  The -nimage is available in some of QuantumESPRESSO codes, like `ph.x` and `neb.x` dividing processors into groups assigned to weakly communicating computation images.

Next parameters control the parallel execution of `pw.x`. For example, launching `pw.x` with the following configuration,

```bash
srun pw.x -nk 32 -nt 4 -nd 4 -i my.input
```

distributed 512 CPUs across 32 pools of k-points with 16 processors each. Each pool of 16 processors is split to four FFT tasks (`-nt` flag). Finally, the Hamiltonian matrix is distributed over 4 CPUs (`-nd` flag, recommended when N_bands is few hundred or more). More detailed explanations are found in the corresponding 'Parallel execution' section in the QuantumESPRESSO [user manual](https://www.quantum-espresso.org/Doc/pw_user_guide/node19.html).
