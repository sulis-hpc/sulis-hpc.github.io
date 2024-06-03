---
layout: page
title: Resource limits
parent: Job submission
grand_parent: Getting Started
nav_order: 8
---

# Resource Limits
{: .no_toc }

1. TOC
{:toc}

Some of the resource limits we have in place are based on the hardware configuration of the compute nodes. For example, the maximum amount of memory a job can request on a compute node is just below the amount of physical RAM in that node to ensure that the operating system has enough RAM to function and not result in the node running out of memory.

Some of the resource limits we have in place (such as the walltime and job size limits) are in place to ensure that the available resources are fairly shared between users. Such limits may be reviewed and amended from time to time.

## Per node limits

| Limit | Compute | High Memory | Very High Memory | GPU |
|-------|:-------:|:-----------:|:----------------:|:---:|
| max no. of cores per node | 128 | 128 | 64| 128 |
| max memory per core | 3,850 MB | 7,700 MB | 64,000 MB | 3,850 MB |
| max memory per node | 492,800 MB | 985,600 MB | 4,096,000 MB | 492,800 MB |
| max gpus per node | N/A | N/A | N/A | 3 A100s |

## Per Partition Limits

Where low priority partitions exist, differences to the standard limits are indicated in parentheses. 

| Limit | compute | devel | gpu | gpu-devel | hmem | vhmem |
|-------|:-------:|:-----:|:---:|:---------:|:----:|:-----:|
| max walltime | 48 hours | 1 hour | 48 (24) hours | 1 hour | 48 hours | 48 hours |
| max cores per job | 3840 | 256 | 1290 | 128 | 384 | 192 |
| max cores per user | 7680 | 256 | 1290 | 128 | 384 | 192 |
| max gpus per job | N/A | N/A | 30 | 3 | N/A | N/A |
| max gpus per user | N/A | N/A | 30 | 3 | N/A | N/A |
| max running jobs (per user) | 200  | 2 | 30 (10) | 2 | 100 | 100 |
| max jobs in queue (per user) | 500 | 4 | 200 (50) | 4 | 200 | 200 |

