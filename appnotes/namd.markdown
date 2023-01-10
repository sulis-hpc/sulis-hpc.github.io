---
layout: page
title: NAMD
parent: Application Notes
nav_order: 6
---

# NAMD on Sulis

NAMD is a parallel molecular dynamics code designed for high-performance simulation of large biomolecular systems.


## Accessing NAMD

List the available NAMD modules:

```bash
[user@login01(sulis) ~]$ module spider NAMD
```

List the details and requirements for a particular NAMD module from that list:

```bash
[user@login01(sulis) ~]$ module spider NAMD/2.14-mpi
```

Load the module, after loading the requirements:

```bash
[user@login01(sulis) ~]$ module load GCC/10.2.0 OpenMPI/4.0.5 NAMD/2.14-mpi
```


## Running NAMD on CPUs


```bash
#!/bin/bash
#SBATCH --time=10:00:00
#SBATCH --job-name=NAMD
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=128
#SBATCH --account=suXXX-somebudget

module purge
module load GCC/10.2.0
module load OpenMPI/4.0.5
module load NAMD/2.14-mpi

charmrun +p$SLURM_NTASKS namd2 +setcpuaffinity stmv.namd
```


The `+setcpuaffinity` flag significantly improves performance.




## Running NAMD on GPUs

This example is for a single GPU job, using 20 CPU cores in support:

```bash
#!/bin/bash
#SBATCH --time=10:00:00
#SBATCH --job-name=NAMD
#SBATCH --partition=gpu
#SBATCH --gres=gpu:ampere_a100:1
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=42
#SBATCH --account=suXXX-somebudget

module purge
module load GCC/10.2.0
module load CUDA/11.1.1
module load OpenMPI/4.0.5
module load NAMD/2.14

charmrun +p20 namd2 +devices $CUDA_VISIBLE_DEVICES +setcpuaffinity stmv.namd
```

Performance will vary with the number of host cores used per-GPU.  Allowing 20 host cores per-GPU is typically a good starting point.



## Performance


The STMV and ApoA1 test cases were downloaded from the [NAMD benchmarking web page](https://www.ks.uiuc.edu/Research/namd/benchmarks/). Please note that these are the basic test cases and not the customised cases used for most of the graphs on that page.

These NAMD 2.14 results are provided as a guide to scaling and to CPU / GPU selection.


### CPU runs

#### STMV 1M atoms

| Number of nodes |1|2|3|4|
|--|--|--|--|--|
| Performance (ns/day) | 1.43 | 2.72 | 3.90 | 4.80 |


#### ApoA1 92k atoms

| Number of nodes |1|2|3|4|
|--|--|--|--|--|
| Performance (ns/day) | 13.4 | 19.6 | 28.3 | 27.3 |





### GPU runs

#### STMV 1M atoms

| Number of GPUs | 1 | 2 | 3 |
|--|--|--|--|
| ns/day | 3.25 | 4.96 | 5.62 |


#### ApoA1 92k atoms

| Number of GPUs | 1 | 2 | 3 |
|--|--|--|--|
| ns/day | 35.3 | 45.7 | 54.4 |




