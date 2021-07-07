---
layout: page
title: Job submission
parent: Getting Started
has_children: true
nav_order: 5
---

Sulis uses a workload management system known as [SLURM](https://slurm.schedmd.com/overview.html) which will likely be familiar to users of other HPC clusters.

In most cases users interact with SLURM by preparing their calculation/workflow as a non-interactive *job script* which contains a resource request, and a sequence of terminal commands to execute using the allocated resource. The script is then submitted to SLURM and executed when the requested resource is available, provided the user has sufficient resource budget in their account to satisfy the request.

The following pages provide example job scripts for "traditional" HPC workloads, for example multi-threaded code using OpenMP within a single compute node or MPI applications which execute across multiple compute nodes. Example job scripts for computations using a GPU are also given.

Users unfamiliar with batch queue systems or SLURM specifically should start with the [Using SLURM](slurmnotes) section and study the [Single node jobs](singlenode) section which covers the basics of writing a job script and reading SLURM output files.

The [advanced topics section](../../advanced) describes how Sulis can be used to execute high throughput or ensemble computing workloads which consist of many replicas of serial, MPI or OpenMP calculations which may also involve GPUs.