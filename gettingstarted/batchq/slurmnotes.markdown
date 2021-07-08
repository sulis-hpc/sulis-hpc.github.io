---
layout: page
title: Using SLURM
parent: Job submission
grand_parent: Getting Started
nav_order: 1
---

# Using SLURM 
{: .no_toc }

1. TOC
{:toc}

## Terminology

Creation of SLURM submission scripts requires some understanding of the terminology used to refer to elements of the system hardware. It also requires some understanding of how a computational workload is constructed in terms of processes and often threads.

- **Job**: When a user submits a job script to SLURM this creates a job, which is given a unique ID and placed into a queue until the requested resources are available. 

- **Node**: SLURM refers to a single server within the cluster as a *node*. Sulis consists of 167 compute nodes plus 30 nodes equipped with Nvidia A100 GPUs.

- **Partition**: A group of nodes to which job can be submitted, sometimes referred to as a queue for legacy reasons.  The default partition on Sulis is the {{site.data.slurm.cnode_partition_name}} partition which consists of the compute nodes. The GPU-equipped nodes form the {{site.data.slurm.cnode_partition_name}} partition.
<!-- DQ: I know this isn't strictly true. Will we have a devel partition? -->

- **Socket**: SLURM refers to the compute processors (which contain multiple processing cores) by the *socket* they are plugged into. The Sulis nodes each contain two AMD EPYC processors.

- **CPU**: Sulis is configured such that SLURM refers to each processor core as a *CPU*. Each EPYC processor contains 64 processor cores, and hence there are 128 CPUs total per node.

- **Task**: A task represents an instance of a running program/executable. Most jobs suitable for Sulis will involve parallelism using multiple tasks. These tasks may collectively constitute a single calculation using [MPI](https://en.wikipedia.org/wiki/Message_Passing_Interface), or many independent calculations which form part of a single job.

A single job may involve simultaneous execution of tasks across multiple nodes. In turn each task may make use of multiple CPUs via threading (e.g. [OpenMP](https://forum.openmp.org/index.php)) or by spawning child processes (e.g [Python multiprocessing](https://docs.python.org/3/library/multiprocessing.html)).

The maximum number of CPUs that can be used by any one task (without oversubscribing) is the number of CPUs in a node. Jobs which use more than 128 cores will therefore need to use some kind of task-based parallelism.

## Job management using SLURM

To submit a job script to SLURM use `sbatch`.

```shell
{{site.data.terminal.prompt}} sbatch myjob.slurm
```
This adds the job into the queue and returns a unique job ID number.

Jobs in the queue cam be monitored using the `squeue` command. For example to display your own submitted jobs, use: 

```shell
{{site.data.terminal.prompt}} squeue -u username

             JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
            212567   {{site.data.slurm.cnode_partition_name}} CsKAgInC   userna  R    4:27:18      4 {{site.data.hosts.cnode_prefix}}[013,032-034]
            212568   {{site.data.slurm.cnode_partition_name}} CsKAgInC   userna  R    4:27:18      4 {{site.data.hosts.cnode_prefix}}[148-151]
```

The columns in the output are explained as follows:

- **JOBID**: This is the job number assigned to the job when submitted.

- **PARTITION**: This is the partition (or queue) the job was submitted to. If no partition is specified it will automatically go to the "{{site.data.slurm.cnode_partition_name}}" partition.

- **NAME**: This is a name given to the job which by default is the file name of the job script.

- **USER**: Your username. Without the -u option to `squeue` jobs from all users will be listed.

- **ST**: A two letter job state code indicating the current state of the job. The key job state codes are: PD=Pending, R=Running and CG=Completing.

- **TIME**: The time used by the job if it is running in days-hours:minutes:seconds.

- **NODES**: The number of nodes requested or allocated to the job.

- **NODELIST(REASON)**: For pending jobs, this column will show the reason the job is waiting for execution such as PartitionTimeLimit which means that the job has exceeded the maximum time limit of the partition. A full list of Job Reason Codes can be found in the `squeue` man page by running `man squeue` at the command line (use the cursor keys to scroll through the documentation and press q to exit). For jobs that are running, this column will show the list of nodes that have been allocated to a job. 

Please contact [support](../../support/) if you need advice as to why a job is not running. 

Output of `squeue` can be restricted to only pending jobs with 

```shell
{{site.data.terminal.prompt}} squeue -u username -t PD
```

or just running jobs with:

```shell
{{site.data.terminal.prompt}} squeue -u username -t R
```

Jobs can be removed from the queue with `scancel`

```shell
{{site.data.terminal.prompt}} scancel job_id
```

where `job_id` is the uniq ID number SLURM assigned to the job.

