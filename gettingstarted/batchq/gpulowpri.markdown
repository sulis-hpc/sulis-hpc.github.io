---
layout: page
title: Low priority jobs
parent: Job submission
grand_parent: Getting Started
nav_order: 8
---

# Low priority jobs
{: .no_toc }

1. TOC
{:toc}

Users who have exhausted their [resource budget](budgets) for the current allocation period may be able to run jobs on a low priority basis
if the Sulis hardware is not fully utilised and servers are available. Currently this is enabled only for GPU jobs. Information on submitting low priority
GPU jobs which do not consume GPU resource budget is given below.

## Submitting low priority GPU jobs

In order to run low priority GPU jobs you must first have been given permission to do this by whoever manages your project via SAFE. See the
[support page](../../support) for information on who to contact for this to be enabled on your user account. 

Jobs should be submitted to the `gpulowpri` partition rather than the `gpu` partition. In additional they should specify a special budget code
rather than their own GPU budget code. Each HPC Midlands+ site project and GPU-using EPSRC Access to HPC project should have such a budget code
named `suxxx-gpulowpri` where `xxx` is your SAFE project number. An example job submission script is below.

<p class="codeblock-label">gpu.slurm</p>
```bash
#!/bin/bash
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=42
#SBATCH --mem-per-cpu={{site.data.slurm.gpunode_ram_per_core}}
#SBATCH --gres=gpu:{{site.data.slurm.gpunode_gpu_gres_name}}:1
#SBATCH --partition={{site.data.slurm.gpulowpri_partition_name}}
#SBATCH --time=08:00:00
#SBATCH --account=suxxx-gpulowpri

module purge
module load {{site.data.software.defaultgcc}} {{site.data.software.defaultcuda}}

srun ./a.out
```

Other examples from the [GPU jobs](gpu) page can be modified accordinly to use run low priority GPU jobs.

It is also possible to
submit [interactive jobs](interactive) to the `{{site.data.slurm.gpulowpri_partition_name}}` to work in (for example) [Jupyter notebooks](../../appnotes/jupyter)
without consuming any GPU resource budget.

## Resource limits

In order to limit the time which higher priority users (those with remaining budget) spend queueing behind lower priority jobs, there are
some restrictions on the use of the low priority GPU partition compard to the standard GPU partition.

- The walltime limit is shorter (24 hours rather than 48)
- There maximum number of queued and running jobs per user is smaller (50 and 10 respectively). 

Other limits are the same as for the standard `gpu` partition documented on the [resource limits page](limits). 


## Notes for SAFE project managers

Low priority GPU jobs consume a special class of resource named `SulisLowPri` within SAFE. Each top-level Sulis project
that needs access to GPUs should have a SAFE project group named `suxxx-gpulowpri` where `xxx` is the SAFE Sulis project number.
This contains an essentially infinite amount of `SulisLowPri` resource in an allocation that runs until the anticipated end
date of the Sulis service. Users should be able to run jobs against this budget indefinately without exhausting it.

Project managers should be aware of two details.

1. Users will need to be added as members of the appropriate `suxxx-gpulowpri` project group in SAFE by their project
manager in order to use the `{{site.data.slurm.gpulowpri_partition_name}}` partition. This is left to the discretion of project managers and PIs who
may wish to restrict access at the project/institutional level.

2. As with standard `SulisGPU` resource and the `{{site.data.slurm.gpunode_partition_name}}` SLURM partition, the `suxxx-gpulowpri` project group must contain
a positive quantity of `SulisCPU` resource. This will not be consumed by running jobs in the `gpulowpri` partition,
but must nonetheless be present in order for jobs to start. Jobs accidentaly submitted to the `{{site.data.slurm.cnode_partition_name}}` partition
against a `suxxx-gpulowpri` budget *will* consume this CPU resource. This should be avoided. 

