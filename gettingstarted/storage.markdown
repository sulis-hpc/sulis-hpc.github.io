---
layout: page
title: Storage 
parent: Getting Started
nav_order: 3
---

# Files and storage 
{: .no_toc }

1. TOC
{:toc}

## Home directory

Sulis contains a dedicated high performance storage system utilising the IBM General Parallel File System (GPFS); the brand name of GPFS is IBM Spectrum Scale and you may have heard it referred to as this elsewhere. 

User home directories on Sulis are intended to provide working space for current jobs only and therefore it is the expectation that users will move their data to other systems once the data is no longer needed on the cluster. Accordingly there is **NO BACKUP OR DISASTER RECOVERY** for GPFS and in the event of major hardware failure, accidental data deletion, file system corruption etc. the data will be lost.

Please consult with the local research computing support team at your home university/institute regarding options for long-term and secure storage of data generated on the Sulis HPC service.

The path to your home directory on each cluster is: 

```
/gpfs/home/<letter>/<username>
```

Where `<username>` is the name of your machine account on Sulis and `<letter>` is the first letter of that name.


## Quota

Your GPFS home directory is subject to a quota of several TB and several million files/directories (inodes) with seven days of grace time. 

Use the mmlsquota command to determine your specific quota level and usage: 

```
{{site.data.terminal.prompt}} mmlsquota --block-size auto
```

## Scratch filesystem

Users with experience of other HPC facilities may be aware of the concept of a cluster-wide "scratch file system". The purpose of the scratch file system is often to provide a location where jobs can read or write their data to/from during a compute job so that the home directories are not affected by the "load" on the GPFS storage system. 

Sulis has a limited amount of cluster wide scratch space on SSDs mounted as `/scratch/`. This scratch space may provide a performance boost for certain read/write intensive workloads as compared to the home disk area.

Due to the limited capacity of this scratch area, access is by request only. When applying for access to Sulis via your institution or EPSRC calls, please indicate if you anticipate a need to use this storage. 

Do please note that `/home/` is by no means low-performance, especially when utitising applications with parallel IO, and that individual compute nodes have their own local SSDs for temporary storage in `/tmp/`. The `/scratch/` space is hence most appropriate for IO-intensive workloads which require retention of data between job submissions to the batch queue.