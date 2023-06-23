---
layout: page
title: Resource budgets 
parent: Job submission
grand_parent: Getting Started
nav_order: 2
---

# Account budgets 
{: .no_toc }

1. TOC
{:toc}

## SAFE Budgets

All SLURM jobs on Sulis are charged against an account budget of CPU or GPU resource via SAFE. All users of Sulis will be members of a SAFE project. Projects consist or one or more *project groups* to which CPU and GPU resource can be allocated from the overall project budget.

- For HPC Midlands+ users there is one SAFE project per institution. Budget will be allocated to project groups by your local research computing team from the overall budget allocated to the project.

- EPSRC Access to HPC projects will receive a dedicated SAFE project. Resource within the project can be allocated to project groups by the designated project manager, usually the PI of the project.

Jobs can only start if there is a positive budget in the account specified by the job script. Note that jobs will run to completion even if the budget needs more CPU or GPU resource than the budget contains when the job starts. This can lead to large negative budgets if large (or many small) jobs start when the budget is low, but still positive.

## SAFE Allocation periods

Projects are allocated a quanity of CPU and GPU hours to be used within a particular time interval. These intervals are known as *allocation periods* in SAFE. 

Sulis allocation periods will normally start just after midnight on the first day of a month, and finish at midnight on the final day of a month. Any budget remaining at the end of an allocation period will be lost. Jobs that have started running will be allowed to complete if they run past the end of an allocation period. Queued jobs that have not started due to lack of postive budget will run at the start of the next period if there is positive budget for the account in the new period. 

Allocation periods and the overall budget for a project within each allocation period can only be set by the Sulis administration team at Warwick. These periods are set in advance, starting and ending automatically.

### EPSRC Access to HPC projects

Projects allocated via the EPSRC Access to HPC mechanism will receive their budgets in three-month allocation periods. Applicants for CPU or GPU time via this mechanism will be asked to specify how much time is required in each allocation period when completing a technical assessment to submit with their applications. 

### HPC Midlands+ projects

Allocation periods are usually six months in duration. These periods are desynchronised accross the consortium to minimise contention during any end of period rush. 

Some HPC Midlands+ sites allocate budget to project groups for shorter (e.g. three month) intervals rather than the entire SAFE allocation period. If the start or end of these shorter periods does not coincide with that of the overall SAFE allocation period then any changes to project group budgets will not be scheduled in advance by SAFE. The changes will instead take effect immediately at the time the budget reallocations are made manually by the local reseach computing team. Practically, this means that project group budgets are likely to change on a working day close to the start/end of a quarterly period and not exactly at midnight on the last day of a month unless the change corresponds to a new SAFE allocation period for the overall institutional project.

## Querying available budget

The `account-balance` command can be run on the login node to see the current budget names you have access to and their available CPU and GPU resource for the current allocation period:

```bash
{{site.data.terminal.prompt}} account-balance

------------------------------------------------------------
|  SAFE Budget/Account |       CPU hours |       GPU hours |
------------------------------------------------------------
|                suxxx |         531,538 |               0 |
|            suxxx-gpu |               1 |           8,917 |
------------------------------------------------------------
```

The account balance is updated every 30 minutes. Negative balances are not shown here and must be viewed on the [SAFE](https://safe.epcc.ed.ac.uk/) website.

## Specifying the budget when submitting jobs

The budget must be specified for all jobs submitted to Sulis; the first column from the account-balance command contains the budget/account name that should be used when submitting jobs to Sulis. This can be done via the sbatch command line

```bash
{{site.data.terminal.prompt}} sbatch --account=suxxx-somebudget myjob.slurm
```

or within the job script itself by adding

```bash
#SBATCH --account=suxxx-somebudget
```

after the resource request. Here replace `suxxx-somebudget` with the budget code given to you by your local research IT team or project manager.

## GPU nodes

Use of the GPU nodes requires access to appropriate GPU resource budget (measured in GPU-hours rather than CPU hours) _and_ at least 1 CPU hour; this CPU hour will not be consumed by jobs run in the gpu or gpu-devel partitions.
