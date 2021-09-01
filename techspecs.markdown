---
layout: page
title: Technical Specifications
permalink: /techspecs/
nav_order: 7
---

# Sulis Technical Specifications

Based on Dell R-series servers supplied by OCF, Sulis has a total of 25,728 AMD EPYC compute cores and 90 NVIDIA A100 GPUs equivalent to 1.8 double-precision PFLOPS of computing power. The detailed system specifications are as follows:

* Compute nodes: Dell PowerEdge R6525 compute nodes each with 2 x AMD EPYC 7742 (Rome) 2.25 GHz 64-core processors; 128 cores per node; 167 nodes; 21,376 compute cores; 512 GB DDR4-3200 RAM per node; 4 GB RAM per core
* GPU nodes: Dell PowerEdge R7525 nodes each with 3 x NVIDIA A100 40 GB RAM passively-cooled GPUs; 2 x AMD EPYC 7742 (Rome) 2.25 GHz 64-core processors per node; 30 nodes; 90 GPUs; 3,840 CPU cores; 512 GB DDR4-3200 RAM per node; 4 GB per core
* High-memory nodes: Dell PowerEdge R6525 nodes each with 2 x AMD EPYC 7742 (Rome) 2.25 GHz 64-core processors; 128 cores per node; 4 nodes; 512 cores; 1 TB DDR4-3200 RAM per node; 8 GB RAM per core
* Interconnect: Mellanox ConnectX-6 HDR100 (100 Gbit/s) InfiniBand
* Storage: 2 PB IBM Spectrum Scale (GPFS)
* OS: CentOS 8.x
* Batch system: Slurm Workload Manager
