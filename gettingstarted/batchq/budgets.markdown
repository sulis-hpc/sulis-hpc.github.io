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

All SLURM jobs on Sulis are charged against an account budget of CPU and GPU resource. All users of Sulis will be members of one or more SAFE projects and have access to a budget within that project allocated by the project managers.

- For HPC Midlands+ users there is one SAFE project per institution. Budgets within each project will be set by the local research computing team.

- Projects outside of the HPC Midlands+ consortium will receive a dedicated SAFE project. Resource within the project can be allocated to budgets by the designated project manager.

## Querying available budget

The `account-balance` command can be run on the login node to see the current budget names you have access to and their available CPU and GPU resource:

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
