---
layout: page
title: Technical Specifications
permalink: /techspecs/
nav_order: 6
---

# Sulis Technical Specifications

Based on Dell R-series servers supplied by OCF, Sulis has a total of 28,244 AMD EPYC compute cores, 90 NVIDIA A100 GPUs and 54 NVIDIA L40 GPUs. The detailed system specifications are as follows:

* Compute nodes: Dell PowerEdge R6525 compute nodes each with 2 x AMD EPYC 7742 (Rome) 2.25 GHz 64-core processors; 128 cores per node; 167 nodes; 21,376 compute cores; 512 GB DDR4-3200 RAM per node; 4 GB RAM per core
* A100 GPU nodes: Dell PowerEdge R7525 nodes each with 3 x NVIDIA A100 40 GB RAM passively-cooled GPUs; 2 x AMD EPYC 7742 (Rome) 2.25 GHz 64-core processors per node; 30 nodes; 90 GPUs; 3,840 CPU cores; 512 GB DDR4-3200 RAM per node; 4 GB per core
* L40 GPU nodes: Dell PowerEdge R7525 nodes each with 3 x NVIDIA L40 48 GB RAM passively-cooled GPUs; 2 x AMD EPYC 7763 (Milan) 2.45 GHz, 64 core processor per node; 18 nodes; 54 GPUs; 2304 CPU cores; 512 GB DDR4-3200; 4 GB per core
* High-memory nodes: Dell PowerEdge R6525 nodes each with 2 x AMD EPYC 7742 (Rome) 2.25 GHz 64-core processors; 128 cores per node; 4 nodes; 512 cores; 1 TB DDR4-3200 RAM per node; 8 GB RAM per core
* Very High-memory nodes: Dell PowerEdge R6525 nodes each with 2 x AMD EPYC 7542 (Rome) 2.25 GHz 32-core processors; 64 cores per node; 3 nodes; 192 cores; 4 TB DDR4-3200 RAM per node; 64 GB RAM per core
* Interconnect: Mellanox ConnectX-6 HDR100 (100 Gbit/s) InfiniBand
* Storage: 2.6 PB (HDD) and 800TB (SSD) IBM Spectrum Scale (GPFS) 
* Batch system: Slurm Workload Manager
