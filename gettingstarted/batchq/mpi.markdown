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
#SBATCH --account=suxxx-somebudget

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

The MPI4Py Python package is available via the `mpi4py` environment module.

{: .note } 
For older toolchains the MPI4Py Python package was included in the `SciPy-bundle`
environment module rather than being provisioned separately. 


A suitable job script would be the following:

<p class="codeblock-label">mpi4py.slurm</p>
```bash
#!/bin/bash
#SBATCH --nodes=4
#SBATCH --ntasks-per-node={{site.data.slurm.cnode_cores_per_node}}
#SBATCH --mem-per-cpu={{site.data.slurm.cnode_ram_per_core}}
#SBATCH --time=08:00:00
#SBATCH --account=suxxx-somebudget

module purge
module load {{site.data.software.defaultgcc}} {{site.data.software.defaultmpi}}
module load {{site.data.software.mpi4pymodule}}

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
#SBATCH --account=suxxx-somebudget

module purge
module load {{site.data.software.defaultgcc}} {{site.data.software.defaultmpi}}
module load {{site.data.software.mpi4pymodule}}

srun python -m mpi4py.futures example_mpifuture.py 255
```

In the above example each worker uses a single CPU, but it may be appropriate to use multiple CPUs per task if the code to be evaluated by the workers can use multiple threads and releases the Global Interpreter Lock (GIL).

The number of inputs to evaluate needn't match the number of workers, but should be close to an integer multiple of the number of workers if the expected computation time for each evaluation of the function is similar.

## Parallel cluster in R

A convenient way to exploit multinode parallelism in R is to create a "cluster" within an R script which can then be exploited by functions such as `parLapply`. Within a SLURM environment, one way this can be accomplished is via the MPI cluster type and launching the calculation using the RMPISNOW wrapper. 


{: .note } 
Users will need to installed the `Rmpi` and `snow` packages into their R environment. See the [software page on R](../software/R) for information on how to install R packages.


An example R script which illustrates this is below. 

<details markdown="block" class="detail">
  <summary>R script using an MPI "cluster"<code>example_mpi.R</code>.</summary>
This generates N samples from the standard normal distribution, and then performs a bootstrap analysis of the mean by resampling (with replacement) k times from these N samples. The distribution of means is compared to the standard error of the original sample set and the distribution of means is compared to the expected form.

Note that only one of the tasks will execute the master script below. The remaining tasks will act as workers to execute instances of the function `resample`. Any data needed by the workers must be exported to the workers or it will not be available.

Note also that in this example we read N and k from environment variables set the SLURM job script. This is because command line arguments cannot be accessed portably when creating an MPI "cluster" in this way.

<p class="codeblock-label">example_mpi.R</p>
```R
#!/usr/bin/env Rscript

# Create a cluster 
library(Rmpi)
library(snow)
cl <- makeCluster()

# Get N and k from environment variables specified in slurm job script
N <- as.integer(Sys.getenv("N"))
k <- as.integer(Sys.getenv("k"))

# Export to all workers
clusterExport(cl, c("N", "k"))

# Generate N samples from the normal distribution, then do a 
# bootstrap error analysis on the mean with K trials
samples <- rnorm(N)

# Export this data to all workers
clusterExport(cl, "samples")

resample <- function(trial) {
  new_samples <- sample(samples, N, replace=TRUE)
  new_mean <- mean(new_samples)
}

# Conduct trials in parallel using parLapply over the cluster cl
timing <- system.time({
  resampled_means <- unlist(parLapply(cl,1:k, resample))
})

# Print timing
print(timing)

av <- mean(samples)        # mean
se <- sd(samples)/sqrt(N)  # standard error

# Histogram of the K resampled means and expected distribution
hist(resampled_means, prob = TRUE)
curve(dnorm(x, av, se), col = "red", add = TRUE)

# Release the workers
stopCluster(cl)
``` 
</details>

A suitable job submission script which launches the master and worker processes is given below. We set the input variables N and k as environment variable which will be read by the R script.

<p class="codeblock-label">Rmpi.slurm</p>
```bash
#!/bin/bash
#SBATCH --nodes=2
#SBATCH --ntasks-per-node=8
#SBATCH --cpus-per-task=1
#SBATCH --mem-per-cpu={{site.data.slurm.cnode_ram_per_core}}
#SBATCH --time=00:01:00
#SBATCH --account=suxxx-somebudget

module purge
module load {{site.data.software.Rtoolchain}} {{site.data.software.Rmodule}}

# Add tools for MPI cluster to path (assumes default location for user-installed packages)
export PATH=${HOME}/R/x86_64-pc-linux-gnu-library/{{site.data.software.Rshortver}}/snow:$PATH

# Set shell variables read by example_mpi.R as input
export N=10000
export k=50000

# Launch via RMPISNOW script
srun RMPISNOW CMD BATCH example_mpi.R 
```
This will create 1 master and 15 worker processes across 2 nodes. This is for illustrative purposes only. Normally it would not be necessary to split a 16 processor job across two nodes in this way. In principle one can use this method to use all processors across multiple nodes in the Sulis system for workloads that benefit from very large amounts of parallelism. 

## Using `MPI.jl` in Julia

The Julia package [MPI.jl](https://juliaparallel.org/MPI.jl/stable/configuration/) provides an interface between Julia and MPI libraries. This allows code written in Julia to implement message passing parallelism between many running instances of the same Julia program. 

A trivial example using MPI in Julia is below.

<details markdown="block" class="detail">
  <summary>Example MPI.jl code<code>01-hello.jl</code>.</summary>
  Simple hello word example adapted from the ``MPI.jl`` [documentation](https://juliaparallel.org/MPI.jl/stable/examples/01-hello/).


<p class="codeblock-label">01-hello.jl</p>
```julia
using MPI
MPI.Init()

comm = MPI.COMM_WORLD
println("Hello world, I am MPI rank $(MPI.Comm_rank(comm)) of $(MPI.Comm_size(comm))")
MPI.Barrier(comm)
``` 
</details>

To use `MPI.jl` on Sulis users must load both a Julia module and an MPI module. For example:

```bash
{{site.data.terminal.prompt}} module load Julia/{{site.data.software.juliamodver}}
{{site.data.terminal.prompt}} module load {{site.data.software.defaultfoss}}
```

Some configuration is required to point `MPI.jl` to the OpenMPI libraries provided by the module. This involves setting the ``JULIA_DEPOT_PATH`` variable as described in the [Julia section of the software pages](../software/julia.markdown#installing-julia-packages).

```bash
{{site.data.terminal.prompt}} export JULIA_DEPOT_PATH=~/julia
{{site.data.terminal.prompt}} julia -e 'using Pkg; Pkg.add("MPIPreferences"); Pkg.add("MPI")'
{{site.data.terminal.prompt}} julia -e 'using MPIPreferences; MPIPreferences.use_system_binary()'
```

You should see output that identifies the version of MPI provided by the ``OpenMPI`` module loaded above.

```
    ┌ Info: MPI implementation identified
    │   libmpi = "libmpi"
    │   version_string = "Open MPI v{{site.data.software.ompiver}}, package: Open MPI Distribution, ident: {{site.data.software.ompiver}}, repo rev: v{{site.data.software.ompiver}}, May 26, 2022\0"
    │   impl = "OpenMPI"
    │   version = v"{{site.data.software.ompiver}}"
    └   abi = "OpenMPI"
    ┌ Info: MPIPreferences changed
    │   binary = "system"
    │   libmpi = "libmpi"
    │   abi = "OpenMPI"
    │   mpiexec = "mpiexec"
    │   preloads = Any[]
    └   preloads_env_switch = nothing
```

With the prerequisites satisfied we can submit a job script which loads the appropriate 
module and launches programs which use MPI.jl. An example script follows which launches
the example ``01-hello.jl`` from the [MPI.jl documentation](https://juliaparallel.org/MPI.jl/stable/examples/01-hello/) on 4 processors. 

<p class="codeblock-label">Rmpi.slurm</p>
```bash
#!/bin/bash
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=4
#SBATCH --cpus-per-task=1
#SBATCH --mem-per-cpu={{site.data.slurm.cnode_ram_per_core}}
#SBATCH --time=00:01:00
#SBATCH --account=suxxx-somebudget

module purge
module load Julia/{{site.data.software.juliamodver}}
module load {{site.data.software.defaultfoss}}
    
# Make sure our extra packages are available
export JULIA_DEPOT_PATH=~/julia

srun julia 01-hello.jl 
```

The number of tasks can be increased up to {{site.data.slurm.cnode_cores_per_node}} on a single node on the {{site.data.slurm.cnode_partition_name}} partition, and beyond this by increasing the ``--nodes`` part of the resource request to 2 or more nodes. As with all parallel calculations, users should experiment to find the optimal amount of CPU resource run their calculation efficiently. Throwing as many CPUs at the calculation as possible and hoping for the best is unlikely to be effective. Communication costs between tasks can more than offset any benefit of distributing work over additional tasks. 


## Under-populating nodes

Some codes can have very intensive memory requirements and require more than {{site.data.slurm.cnode_ram_per_core}} MB of RAM per task. It may therefore be desirable to under-populate nodes with fewer tasks than the available cores. This can be accomplished in two ways. For example to launch 64 tasks per node with access to 2x{{site.data.slurm.cnode_ram_per_core}} MB per task either;

1. change the resource request portion of the script to;
```bash
#!/bin/bash
#SBATCH --nodes=4
#SBATCH --ntasks-per-node=64
#SBATCH --cpus-per-task=2
#SBATCH --mem-per-cpu={{site.data.slurm.cnode_ram_per_core}}
#SBATCH --time=08:00:00
#SBATCH --account=suxxx-somebudget
````
2. leave the resource request unchanged but launch the MPI program with:
```bash
srun -n 64 -c 2 ./a.out
````

In either case half of the CPU cores in the node will be left idle, but each task will access to twice as much RAM. This assumes the MPI program cannot make use of multiple cores per task. See also [hybrid jobs](../hybrid/). 

Jobs which under-populate nodes will be charged against resource budgets as if all cores in the node were fully utilised, so this should only be done if absolutely necessary. 

{: .note } 
Sulis contains a small number of servers with larger amounts of RAM, accessed via the `{{site.data.slurm.vfatnode_partition_name}}` and `{{site.data.slurm.vfatnode_partition_name}}` partitions. See the [high memory jobs](highmem) page for details.