---
layout: page
title: Hybrid jobs
parent: Job submission
grand_parent: Getting Started
nav_order: 5
---

# Hybrid jobs
{: .no_toc }

1. TOC
{:toc}

Hybrid jobs are those which involve running multiple tasks, each of which make use of multiple CPUs via threading. This will usually be MPI C/C++ or Fortran code with threading implemented using OpenMP pragmas/directives.

Multithreaded jobs are less common in a Python context due to the Global Interpreter Lock (GIL) which prevents multiple python threads in a task from executing concurrently. In some cases though use of multiple threads in Python can be beneficial if execution time is dominated by time spent in Python packages which release the GIL, such as NumPy. An example is given below.


## OpenMP + MPI

A trivial example of a hybrid OpenMPI+MPI code is given below.

<details markdown="block" class="detail">
  <summary>An example OpenMP+MPI program in C.</summary>
An trivial example of a hello world hybrid code.

<p class="codeblock-label">hybrid_hello.c</p>
```c 
#include <stdio.h>
#include <stdio.h>
#include <stdlib.h>
#include "mpi.h"
#include "omp.h"

int main(int argc, char* argv[]) {

  int my_rank, nthreads, tid;

  /* Set a level of required threading support */
  int required = MPI_THREAD_FUNNELED, provided;

  /* Initialise MPI and see if we can get that support*/
  MPI_Init_thread(&argc, &argv, required, &provided);

  /* Get the rank of the current process */
  MPI_Comm_rank ( MPI_COMM_WORLD, &my_rank );

  /* See what we've got and ignore at your peril */
  if ( required != provided ) {
    if (my_rank==0) printf("Sorry - required thread support not available\n");
    MPI_Barrier(MPI_COMM_WORLD);
    MPI_Finalize();
    exit(EXIT_FAILURE);
  }

  /* Begin OpenMP parallel region */
#pragma omp parallel default(shared) private(tid,nthreads) 
{

  /* All threads set a value of the reduction variable */
  tid = omp_get_thread_num();
  nthreads = omp_get_num_threads(); 
  printf("Hello from MPI rank %d thread number %d \n", my_rank,tid);

} /* End OMP parallel region */

  /* Shut down MPI */
  MPI_Finalize();
  return(EXIT_SUCCESS);

}

``` 
To compile this we need to use the appropriate MPI compiler wrapper and pass the appropriate
compiler flag for OpenMP compilation.
```bash
{{site.data.terminal.prompt}} module load {{site.data.software.defaultfoss}}
{{site.data.terminal.prompt}} mpicc hybrid_hello.c -fopenmp
```
</details>

When making a resource request for a hybrid job, the product of `ntasks-per-node` and `cpus-per-task` should equal {{site.data.slurm.cnode_cores_per_node}} as in the following example which launches 32 tasks of 8 threads on each of 2 nodes. Note that we set the environment variable OMP_NUM_THREADS explicitly from information provided by SLURM such that tasks do not oversubscribe CPUs with threads.

<p class="codeblock-label">hybrid.slurm</p>
```bash
#!/bin/bash
#SBATCH --nodes=2
#SBATCH --ntasks-per-node=16
#SBATCH --cpus-per-task=8
#SBATCH --mem-per-cpu={{site.data.slurm.cnode_ram_per_core}}
#SBATCH --time=08:00:00

module purge
module load {{site.data.software.defaultgcc}} {{site.data.software.defaultmpi}}

export OMP_NUM_THREADS=$SLURM_CPUS_PER_TASK

srun ./a.out
```

Determining the optimal split between tasks and threads for a hybrid code can be a complex task. In many cases using {{site.data.slurm.cnode_cores_per_node}} threads per task will be too many as the OpenMP parallel part of the code will not scale effectively to a whole node. Users should consult the manual or developer of the code they are using and experiment to find the optimal performance. 

## MPI4Py + NumPy threads

As per above, if the workload of a Python process is dominated by operations which release the GIL then hybrid parallelism with MPI and threading may be valuable. The following example demonstrates this, setting the number of threads to use within NumPy according to an environment variable
set by SLURM.

<details markdown="block" class="detail">
  <summary>MPI4Py + threaded NumPy example <code>mpi_np_threads.py</code>.</summary>
In this example each MPI task constructs an two random NxN matrices and multiples them together. 

<p class="codeblock-label">mpi_np_threads.py</p>
```python
from mpi4py import MPI
import numpy as np
from threadpoolctl import threadpool_limits 
import os

comm = MPI.COMM_WORLD
my_rank = comm.Get_rank()
p = comm.Get_size()

nt = int(os.environ['SLURM_CPUS_PER_TASK']) # num threads per task

size = 15000 # Matrix size 

with threadpool_limits(limits=nt, user_api='openmp'):

    A = np.random.rand(size, size)
    B = np.random.rand(size, size)

    C = np.matmul(A, B)


MPI.Finalize()
``` 
Note the method used here to set the number of threads used by NumPy is specific to 
builds of NumPy which use OpenMP for threading. This is true for the version of NumPy
made available to Python within the example job script below. 
</details>

The following script would be used to run an MPI4Py python script on a single node, with 32 MPI tasks each using 4 threads. 

<p class="codeblock-label">hybrid_numpy.slurm</p>
```bash
#!/bin/bash
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=32
#SBATCH --cpus-per-task=4
#SBATCH --mem-per-cpu={{site.data.slurm.cnode_ram_per_core}}
#SBATCH --time=08:00:00

module purge
module load {{site.data.software.defaultgcc}} {{site.data.software.defaultmpi}}
module load {{site.data.software.defaultscipy}}


srun python mpi_np_threads.py
```

In this configuration the 32 matrix multiplications are executed approximately 2.4 times faster using 4 threads each compared to when using a single thread. Note that for this particular example it would be necessary to allocate multiple CPUs to each task in order for each MPI rank to have access to enough RAM to execute matrix multiplications of this size. Any speedup gained by making use of these additional CPUs via threading is therefore worthwhile.


