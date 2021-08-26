---
layout: page
title: MPI jobs 
parent: Job submission
grand_parent: Getting Started
nav_order: 4
---

# MPI jobs
{: .no_toc }

1. TOC
{:toc}

The most common way for calculations to make use of multiple nodes in a cluster involves multiple tasks communicating via message passing using MPI. Many simulation packages are written to use MPI.

Other methods for distributing calculations over multiple nodes may use MPI as a backend to benefit from the high performance interconnect in HPC clusters.

## Compiled MPI codes

See the [compiling section](../software/compiling) for information on compiling MPI codes on Sulis.

<details markdown="block" class="detail">
  <summary>An example MPI program in C.</summary>
An trivial example of a hello world code in MPI.

<p class="codeblock-label">mpi_hello.c</p>
```c 
#include <stdio.h>
#include <stdio.h>
#include <stdlib.h>
#include "mpi.h"

int main(int argc, char* argv[]) {

  int my_rank, p;
  MPI_Init(&argc, &argv);
  MPI_Comm_rank ( MPI_COMM_WORLD, &my_rank );
  MPI_Comm_size ( MPI_COMM_WORLD, &p );
  printf("Hello from task %d of %d\n", my_rank, p );
  MPI_Finalize();
  exit(EXIT_SUCCESS);

}
``` 
This might be compiled into the executable `a.out` via:
```bash
{{site.data.terminal.prompt}} module load {{site.data.software.defaultfoss}}
{{site.data.terminal.prompt}} mpicc mpi_hello.c
```
</details>

The following example would launch a total of 512 tasks across 4 whole nodes. By default
srun will launch as many tasks as specified in the job script resource request.

<p class="codeblock-label">mpi.slurm</p>
```bash
#!/bin/bash
#SBATCH --nodes=4
#SBATCH --ntasks-per-node={{site.data.slurm.gpunode_cores_per_node}}
#SBATCH --mem-per-cpu={{site.data.slurm.cnode_ram_per_core}}
#SBATCH --time=08:00:00

module purge
module load {{site.data.software.defaultfoss}}

srun ./a.out
```

## MPI4Py 

The Python interface to MPI can be a useful way to distribute high-level tasks over multiple nodes in a cluster within user code.

<details markdown="block" class="detail">
  <summary>An example MPI program in Python <code>mpi_hello.py</code>.</summary>
The same trivial example from above but now in Python.

<p class="codeblock-label">mpi_hello.py</p>
```python
from mpi4py import MPI

comm = MPI.COMM_WORLD
my_rank = comm.Get_rank()
p = comm.Get_size()

print("Hello from task %s of %s" % (my_rank, p) )

MPI.Finalize()
``` 
</details>

The MPI4Py Python package is included in the SciPy-bundle module. A suitable job script would be the following:

<p class="codeblock-label">mpi4py.slurm</p>
```bash
#!/bin/bash
#SBATCH --nodes=4
#SBATCH --ntasks-per-node={{site.data.slurm.cnode_cores_per_node}}
#SBATCH --mem-per-cpu={{site.data.slurm.cnode_ram_per_core}}
#SBATCH --time=08:00:00

module purge
module load {{site.data.software.defaultgcc}} {{site.data.software.defaultmpi}}
module load {{site.data.software.defaultscipy}}

srun python mpi_hello.py
```
## mpi4py.futures

For Python code which uses a pool of processes via [concurrent.futures](https://docs.python.org/3/library/concurrent.futures.html) (see the [Single Node](singlenode) section), MPI4Py provides a convenient means to extend this over multiple nodes.

<details markdown="block" class="detail">
  <summary>Python program using mpi4py.futures <code>example_mpifuture.py</code>.</summary>
This Python script uses mpi4py.futures to evaluate the function `f(x)` for `N` input values concurrently. N is passed as an argument to the program.
<p class="codeblock-label">example_mpifuture.py</p>
```python
import sys
from mpi4py.futures import MPIPoolExecutor
    
def f(x):
    return x*x

if __name__ == '__main__':

    # This code will be executed on only 1 task

    if len(sys.argv) != 2:
        print("Usage ", sys.argv[0]," <N>")
        sys.exit()
    else:
        N = int(sys.argv[1])
    
    # Create a list of inputs to the function f
    inputs = range(N)
    
    # Evaluate f for all inputs using the pool of processes
    with MPIPoolExecutor() as executor:
        results = executor.map(f, inputs)


    print([result for result in results])
``` 
</details>

The necessary submission script launches multiple tasks on each node. One task is used to run the master Python script and the remaining tasks make up the worker pool. In this case we run the above example to evaluate 255 inputs on 255 workers.

<p class="codeblock-label">mpi4py.slurm</p>
```bash
#!/bin/bash
#SBATCH --nodes=2
#SBATCH --ntasks-per-node={{site.data.slurm.cnode_cores_per_node}}
#SBATCH --mem-per-cpu={{site.data.slurm.cnode_ram_per_core}}
#SBATCH --time=08:00:00

module purge
module load {{site.data.software.defaultgcc}} {{site.data.software.defaultmpi}}
module load {{site.data.software.defaultscipy}}

srun python -m mpi4py.futures example_mpifuture.py 255
```

In the above example each worker uses a single CPU, but it may be appropriate to use multiple CPUs per task if the code to be evaluated by the workers can use multiple threads and releases the Global Interpreter Lock (GIL).

The number of inputs to evaluate needn't match the number of workers, but should be close to an integer multiple of the number of workers if the expected computation time for each evaluation of the function is similar.

## Under-populating nodes

Some codes can have very intensive memory requirements and require more than {{site.data.slurm.cnode_ram_per_core}} MB of ram per task. It may therefore be desirable to under-populate nodes with fewer tasks than the available cores. This can be accomplished in two ways. For example to launch 64 tasks per node with access to 2x{{site.data.slurm.cnode_ram_per_core}} MB per task either;

1. change the resource request portion of the script to;
```bash
#!/bin/bash
#SBATCH --nodes=4
#SBATCH --ntasks-per-node=64
#SBATCH --cpus-per-task=2
#SBATCH --mem-per-cpu={{site.data.slurm.cnode_ram_per_core}}
#SBATCH --time=08:00:00
````
2. leave the resource request unchanged but launch the MPI program with:
```bash
srun -n 64 -c 2 ./a.out
````

In either case half of the CPU cores in the node will be left idle, but each task will access to twice as much RAM. This assumes the MPI program cannot make use of multiple cores per task. See also [hybrid jobs](../hybrid/). 

Jobs which under-populate nodes will be charged against resource budgets as if all cores in the node were fully utilised, so this should only be done if absolutely necessary. 

Note that a small number of Sulis nodes have access to {{site.data.slurm.fatnode_ram_per_core}} MB memory per CPU and are available for large memory jobs on request.