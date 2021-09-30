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

## Default budgets

Every user has a default budget to which all SLURM jobs will be charged. For most users this will be the only budget they ever use and there is no need to specify which budget should be used for a particular job. 

The default budget for each user is that to which the associated SAFE account has most recently been associated. Users needing to specify a different budget for a particular job can do so via the sbatch command line

```bash
{{site.data.terminal.prompt}} sbatch --account=suxxx-somebudget myjob.slurm
```

or within the job script itself by adding

```bash
#SBATCH --account=suxxx-somebudget
```

after the resource request. Here replace `suxxx-somebudget` with the budget code given to you by your local research IT team or project manager.

## GPU nodes

Use of the GPU nodes requires access to appropriate GPU resource budget (measured in GPU-hours rather than CPU hours) _and_ a budget of CPU resource. When resource is allocated to projects which need GPU resource, 128 core-hours of CPU budget should be allocated along with every 3 GPU-hours of GPU budget. This ensures GPU jobs are not rejected due to lack of resource to run the CPU (host) code which benefits from the GPU acceleration. 

## Querying available budget

Information on querying budgets from the command line will appear here.