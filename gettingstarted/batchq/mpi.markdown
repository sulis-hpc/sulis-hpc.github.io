---
layout: page
title: MPI jobs 
parent: Job submission
grand_parent: Getting Started
nav_order: 4
---

```bash
#!/bin/bash
#SBATCH --nodes=4
#SBATCH --ntasks-per-node={{site.data.slurm.gpunode_cores_per_node}}
#SBATCH --mem-per-cpu={{site.data.slurm.cnode_ram_per_core}}
#SBATCH --time=08:00:00

module purge
module load {{site.data.software.defaultfoss}}

srun ./a.out
```