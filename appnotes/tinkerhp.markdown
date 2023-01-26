---
layout: page
title: Tinker-HP
parent: Application Notes
nav_order: 10
---

# Tinker-HP on Sulis
{: .no_toc }

1. TOC
{:toc}

[Tinker-HP](https://tinker-hp.org) a Massively Parallel Molecular Dynamics Package for Multiscale Simulations of Large Complex Systems
with Advanced Polarizable Force Fields.

# Accessing Tinker-HP

First, check the list of existing `Tinker-HP` modules

```bash
{{site.data.terminal.prompt}} module spider Tinker-HP
```

The following

```bash
{{site.data.terminal.prompt}} module spider Tinker-HP/1.2
```

Will printout its dependencies to be loaded prior the chosen `Tinker-HP` module. Currently (Jan 2022) there is no GPU version installed on Sulis, but it can be requested.  


# Running Tinker-HP
To run one of the proposed Tinker-HP examples, copy the corresponding files with help of the loaded `Tinker-HP` module

```bash
module purge
module load iccifort/2019.5.281  impi/2019.7.217
module load Tinker-HP/1.2

# copy the example files
cp ${EBROOTTINKERMINHP}/example/{puddle.dyn,puddle.key,puddle.xyz} .

# copy the parameter file
cp ${EBROOTTINKERMINHP}/params/water.prm .

# also point to the copied parameter file in puddle.key
sed -i "s/parameters          ..\/params\/water/parameters          .\/water/"  puddle.key
```

Loaded `Tinker-HP` module is required for defining the `$EBROOTTINKERMINHP` variable, the root directory of the installation. Finally, the following batch script template can be used to submit a job.

```bash
#!/bin/bash
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=12
#SBATCH --cpus-per-task=1
#SBATCH --mem-per-cpu=3850
#SBATCH --time=15:00:00
#SBATCH --account=suXXX-somebudget

ulimit -s unlimited

module purge
module load iccifort/2019.5.281 impi/2019.7.217
module load Tinker-HP/1.2

srun dynamic puddle 10 2.0 1.0 2 300.0
```
