---
layout: page
title: Checkpointing
parent: Advanced Topics
nav_order: 4
---

# Checkpointing
{: .no_toc }

This topic explains how to restart interrupted calculations using the checkpointing software.

1. TOC
{:toc}

## DMTCP
[DMTCP](https://dmtcp.sourceforge.io) is the system-level checkpointing solution allowing to restart an interrupted calculation without any modifications of the code. It is claimed to be able to checkpoint the MPI-parallel applications as well, which is however not the case for the version currently installed on Sulis (2.6.0). It does, however, support the codes compiled with MPI wrappers (e.g., mpicc , mpif90, etc...) being run as a single `MPI` task.

### Basic Usage

<details markdown="block" class="detail">
  <summary>Python code example.</summary>
<p class="codeblock-label">run.py</p>
```python
import time
import numpy as np
start = time.time()
matrix_size=8000
print('Code started')
for i in range(1000):
   start_loop = time.time()
   a = np.random.rand(matrix_size, matrix_size)
   b = np.random.rand(matrix_size, matrix_size)
   c = np.matmul(a,b).sum()
   t_op = time.time()
   template = 'iter, result, time, time per loop: {:4d} {} {:6.2f} {:6.2f}'
   print(template.format(i, c, t_op-start, t_op-start_loop))
```
</details>

The batch submission script below runs the Python program above checkpointed every 60 seconds using the `dmtcp_launch` command and which is also openMP-parallel within the `numpy` linear algebra calls. When interrupted by SLURM after maximal execution time (`--time`), it can be restarted from the most recent checkpoint written into the `ckpt_*.dmtcp` file. Restarting the calculation from this checkpoint is triggered by submitting the same script again.

<p class="codeblock-label">dmtcp.slurm</p>
```bash
#!/bin/bash
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=4
#SBATCH --mem-per-cpu=3850
#SBATCH --time=00:04:00
#SBATCH --account=suXXX-somebudget

# load modules for checkpointing
module purge
module load {{site.data.software.defaultfoss}}
module load DMTCP/{{site.data.software.DMTCPver}}
module load {{site.data.software.defaultpython}}
module load {{site.data.software.defaultscipy}}

# time interval (in seconds) between checkpoint writes
checkpoint_write_interval=60

# check whether a checkpoint file exist
if [ ! -e ckpt*dmtcp ]
then
   echo "checkpoint file not found, starting the app .."
   # launch the app and write a checkpoint each time after $checkpoint_write_interval is over
   # By default, print in Python is buffered, meaning that it does not write to
   # files or stdout immediately, and -u flag changes this behaviour
   srun dmtcp_launch --rm -i $checkpoint_write_interval python -u run.py
else
   echo "restarting the app .."
   # restart from the most recent checkpoint file
   dmtcp_restart -i $checkpoint_write_interval `find . -maxdepth 1 -type f | ls -t ckpt*dmtcp | head -n1`
fi
```
Please note, that using the generated restart script `dmtcp_restart_script.sh` does not always work, and it is therefore recommended restarting the calculations with `dmtcp_restart` as above. There is an overhead for writing checkpoints especially in the case of heavier calculations. Outside the checkpoint writing events, the performance is essentially unaffected, and therefore, increasing the checkpoint write interval improves the overall performance.

Applications compiled with use of `MPI` wrappers can be checkpointed in the same way providing that these are launched as single-node and single-task processes. Finally, `DMTCP` is compiled within several toolchains, and the corresponding information is printed out upon running `module spider DMTCP`.

<!--
```bash
------------------------------------------------------------------------
  DMTCP: DMTCP/3.0.0
------------------------------------------------------------------------
    Description:
      DMTCP is a tool to transparently checkpoint the state of multiple 
      simultaneous applications, including multi-threaded and distributed
      applications. It operates directly on the user binary executable, 
      without any Linux kernel modules or other kernel modifications.


    You will need to load all module(s) on any one of the lines below before
    the "DMTCP/2.6.0" module is available to load.

      GCCcore/10.3.0
      GCCcore/8.3.0
      GCCcore/9.3.0
```
-->
