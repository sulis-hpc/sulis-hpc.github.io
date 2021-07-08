---
layout: page
title: GPU jobs 
parent: Job submission
grand_parent: Getting Started
nav_order: 5
---

Under construction.


## Single GPU 


## Single node, multi-GPU 

Some codes will support use of multiple GPUs out of the box.

In other cases you may need to provide additional information to `srun` to indicate how the available GPUs should be shared across the elements of your calculation.

For example, you may wish to use a whole GPU node to create a Python multiprocessing pool of 18 single-CPU workers which equally share the available 3 GPUs within a node.

srun -n 1 -G 3 -c 18 --cpus-per-gpu=6 python my_thing.py

Alternatively you may have an MPI program in which each of 3 single-CPU tasks can effectively utilise an entire GPU.

srun -n 3 -G 3 -c 1 --gpus-per-task=1 my_mpi_thing

More advanced options give precise control over which tasks/CPUs are allocated to which GPU. See the srun section of the SLURM manual.

## Multi-node GPU jobs
