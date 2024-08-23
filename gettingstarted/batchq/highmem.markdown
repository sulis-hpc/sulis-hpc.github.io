---
layout: page
title: High memory jobs
parent: Job submission
grand_parent: Getting Started
nav_order: 7
---

# High memory jobs
{: .no_toc }

1. TOC
{:toc}

{: .highlight}
There are many scientific workloads that benefit from access to servers
with large amounts of RAM and Sulis is equipped to support these. We do
though frequently encounter cases where requirements for large RAM
are the result of poorly written code or a misunderstanding of inputs to
software. Please do not be offended if we ask to sanity check such things.

## Requesting increased memory with SLURM

Limits on memory are usually derived from the amount of physically
available memory within the node, sometimes with an adjustment to ensure
there is sufficient memory for non-job processes, such as file system
cache. For example, the {{site.data.slurm.cnode_partition_name}} partition 
servers have a total of {{site.data.slurm.cnode_ram_per_node}} per node 
and {{site.data.slurm.cnode_cores_per_node}} cores. After the operating
system and other RAM overheads are considered, this leaves 
{{site.data.slurm.cnode_ram_per_core}} MB per core available to SLURM jobs.

SLURM enforces memory limits such that if a job attempts to use more
memory than it asked for, it will be killed. Typically this results in
out-of-memory error messages, often similar to:

```text
   slurmstepd: error: Detected 1 oom-kill event(s) in step <job_id>.batch cgroup.
```

Jobs will sometimes require more than the default amount of memory per
core. In this case one approach is to request more cores and simply
leave them idle, thereby providing increased effective memory per core.
This is done using SLURM's ``--cpus-per-task`` resource directive. For
example to request {{site.data.slurm.cnode_2x_ram_per_core}} MB for a 
serial job, on the {{site.data.slurm.cnode_partition_name}} partition:


<p class="codeblock-label">extra_ram.slurm</p>
```bash
#!/bin/bash
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=2
#SBATCH --mem-per-cpu={{site.data.slurm.cnode_ram_per_core}}
#SBATCH --time=08:00:00
#SBATCH --account=suxxx-somebudget

module purge
module load {{site.data.software.defaultgcc}}

./a.out
```
In this example, the job is requesting 2 (``--cpus-per-task``) \* {{site.data.slurm.cnode_ram_per_core}}
(``--mem-per-cpu``) = {{site.data.slurm.cnode_2x_ram_per_core}} MB. Whilst two cores are assigned to the job,
provided our executable (``a.out``) runs purely sequential code, one of them will be left idle.

The above method is preferred to requesting 1 CPU with higher ``mem-per-cpu``, as it ensures any CPUs on the server *not* being used by your jobs have the expected {{site.data.slurm.cnode_ram_per_core}} MB per core available to other users. This avoids confusion that arises from servers that appear to be only partly utilised while jobs are queued.

{: .note}
>CPU resource budgets are charged for the number of CPUs requested and for the duration of the job. In the example above
the budget `suxxx-somebudget` *will* be charged 16 CPUh (assuming it runs for the full 8 hours requested) and not 8.
>
>Users should interpret 1CPUh as the cost for accessing 1 core *and* its associated RAM. 


In some cases it may be possible to make some use of the otherwise idle CPU by enabling threading in your software.


## High memory nodes

For jobs that need larger amounts of memory, we have {{site.data.slurm.fatnode_partition_size}} servers available to all users via the `{{site.data.slurm.fatnode_partition_name}}` partition. These have approximately {{site.data.slurm.fatnode_ram_per_node}} (TerraByte) of RAM per server, i.e. twice that of the standard compute nodes. See the [resource limits](limits.markdown) page for details.

They can be accessed for jobs that need large amounts of memory by requesting the `{{site.data.slurm.fatnode_partition_name}}` partition explicitly in your SLURM job script, for example;

<p class="codeblock-label">hmem_job.slurm</p>
```bash
#!/bin/bash
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=2
#SBATCH --mem-per-cpu={{site.data.slurm.fatnode_ram_per_core}}
#SBATCH --partition={{site.data.slurm.fatnode_partition_name}}
#SBATCH --time=08:00:00
#SBATCH --account=suxxx-somebudget

module purge
module load {{site.data.software.defaultgcc}}

./a.out
```
where more memory can be requested by increasing the number of CPUs, up to approximately {{site.data.slurm.fatnode_ram_per_node}}. 

It may be desirable to use the high memory nodes for [interactive jobs](interactive.markdown) in which case add the `{{site.data.slurm.fatnode_partition_name}}` partition to your resource request, e.g.

```bash
{{site.data.terminal.prompt}} salloc --account=suxxx-somebudget -N 1 -n {{site.data.slurm.fatnode_half_cpus}} --mem-per-cpu={{site.data.slurm.fatnode_ram_per_core}} --time=8:00:00
```
requests half the CPUs and half the memory ({{site.data.slurm.fatnode_half_ram}}) on one of the high memory nodes for an interactive job. 

{: .important}
There are only {{site.data.slurm.fatnode_partition_size}} nodes in the `{{site.data.slurm.fatnode_partition_name}}` partition which are often in high demand. Please do not use these nodes for workloads that could be executed on 
the standard compute nodes in the `{{site.data.slurm.cnode_partition_name}}` partition. This will be policed!

## Very high memory nodes

For jobs that need extreme amounts of memory (e.g. metagenomics sequence reconstruction), we have {{site.data.slurm.vfatnode_partition_size}} servers available to all users via the `{{site.data.slurm.vfatnode_partition_name}}` partition. These have approximately {{site.data.slurm.vfatnode_ram_per_node}} (TerraBytes) of RAM per server, i.e. 8x that of the standard compute nodes. See the [resource limits](limits.markdown) page for details.

They can be accessed for jobs that need large amounts of memory by requesting the `{{site.data.slurm.vfatnode_partition_name}}` partition explicitly in your SLURM job script, for example;

<p class="codeblock-label">vhmem_job.slurm</p>
```bash
#!/bin/bash
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=2
#SBATCH --mem-per-cpu={{site.data.slurm.vfatnode_ram_per_core}}
#SBATCH --partition={{site.data.slurm.vfatnode_partition_name}}
#SBATCH --time=08:00:00
#SBATCH --account=suxxx-somebudget

module purge
module load {{site.data.software.defaultgcc}}

./a.out
```
where more memory can be requested by increasing the number of CPUs, up to approximately {{site.data.slurm.vfatnode_ram_per_node}}. 

It may be desirable to use the very high memory nodes for [interactive jobs](interactive.markdown) in which case add the `{{site.data.slurm.vfatnode_partition_name}}` partition to your resource request, e.g.

```bash
{{site.data.terminal.prompt}} salloc --account=suxxx-somebudget -N 1 -n {{site.data.slurm.vfatnode_half_cpus}} --mem-per-cpu={{site.data.slurm.vfatnode_ram_per_core}} --time=8:00:00
```
requests half the CPUs and half the memory ({{site.data.slurm.vfatnode_half_ram}}) on one of the very high memory nodes for an interactive job. 

{: .important}
There are only {{site.data.slurm.vfatnode_partition_size}} nodes in the `{{site.data.slurm.vfatnode_partition_name}}` partition which are often in high demand. Please do not use these nodes for workloads that could be executed on 
the standard compute nodes in the `{{site.data.slurm.cnode_partition_name}}` partition. This will be policed!

