---
layout: page
title: EPOCH
parent: Application Notes
nav_order: 4
---

# EPOCH on Sulis
{: .no_toc }

1. TOC
{:toc}

These notes are a work in progress and do not represent a final position on how to get best performance from EPOCH on Sulis. We anticipate that most use of EPOCH on Sulis will be via workflows that involve many calculations using a single node (or less). We have not studied
multi-node performance in any detail.

## Downloading

EPOCH is not centrally installed on Sulis so you should arrange to have the source code available. EPOCH is almost always used in stock configuration, so please download it from the Warwick Plasma github page at [https://github.com/Warwick-Plasma/epoch](https://github.com/Warwick-Plasma/epoch). You can download a default copy using the command

```terminal
git clone --recursive https://github.com/Warwick-Plasma/epoch.git
```

Note the --recursive flag. This is necessary to download the entire of EPOCH. If you get errors associated with a missing module "SDF" then you didn't use the --recursive flag when you downloaded it.

## Compilation

EPOCH performs best on Sulis using the GFortran/OpenMPI toolchain. You should load the modules GCC/11.2.0  and OpenMPI/4.1.1. There is no substantial difference between different MPI or gfortran versions.

You should then compile EPOCH with

```terminal
make COMPILER=gfortran
```

## Running

EPOCH benefits from pinning of MPI tasks to CPU cores by rank and also makes use of MPI-IO. We recommend launching EPOCH as per the following example job script.

<p class="codeblock-label">epoch.slurm</p>
```bash
#!/bin/bash
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=128
#SBATCH --mem-per-cpu=3850
#SBATCH --time=08:00:00
#SBATCH --account=suxxx-somebudget

module purge
module load GCC/11.2.0 OpenMPI/4.1.1

export OMPI_MCA_io=^ompio

srun --cpu-bind=rank --input {data_dir} ./bin/epoch{n}d
```
Where `{n}` is the dimensionality of EPOCH that you are running
`{data_dir}` should be a file that contains the name of the data directory that contains the input deck that you want to run EPOCH with.
