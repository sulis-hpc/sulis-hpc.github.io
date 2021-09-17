---
layout: page
title: Dask
parent: Ensemble jobs
grand_parent: Advanced Topics
nav_order: 3
---

# Dask
{: .no_toc }

1. TOC
{:toc}


[Dask](https://dask.org) is a framework for parallelism in Python driven workflows. Here we focus on the distributed scheduling functionality in Dask. This can be a powerful way to extend single-node calculations using [concurrent.futures](../../gettingstarted/batchq/singlenode#python-concurrentfutures) or [Joblib](../../gettingstarted/batchq/singlenode#python-joblib) such that the pool of processors spans multiple compute nodes of a HPC system like Sulis. 

## Creating a Dask cluster

Dask operates by creating a "cluster". In this context the term "cluster" refers to a collection of processes (tasks in SLURM) which communicate with each other rather than the physical hardware of the HPC system. A Dask cluster consists of:

- One task for the master Python script. 
- One task for a scheduler, i.e. a process which handles distribution of inputs to workers.
- One task for each of the (potentially very many) workers.

For our purposes a worker is a process which executes one instance of a function at a time, possibly using multiple CPUs. By creating a Dask cluster with many workers, we can process
a large number number of inputs to a function and benefit from parallelism over workers. 

The master Python script must create a client interface to the cluster, which is then available to
perform parallel execution of code.

There is some overhead associated with launching workers and communicating inputs/outputs between
the workers and the scheduler. In general each function to be evaluated by a worker should involve a substantial amount of computation (minutes or hours) rather than seconds. 

The following examples demonstrate two ways to initialise a Dask cluster on Sulis and create a
client interface within Python code. Note that Dask also has functionality to create a cluster which submits separate jobs to SLURM for each worker. This is primarily intended for scenarios in which the Dask workload is required to "fit around" larger jobs. This is not the case on Sulis.

### Using MPI to launch a Dask cluster

When using this method, the master Python script must initialise a
dask cluster using `dask_mpi.initialize()` before creating a client interface. This is illustrated in the example below.

<details markdown="block" class="detail">
  <summary>Creating a Dask cluster using MPI <code>dask_mpi_init.py</code>.</summary>
Minimal Python script which initialises a Dask MPI cluster, creates a client interface to it and reports the number of available workers.

<p class="codeblock-label">dask_mpi_init.py</p>
```python
import os
from dask_mpi import initialize
from dask.distributed import Client

if __name__ == '__main__':

    # Query SLURM environment per worker task
    p = int(os.getenv('SLURM_CPUS_PER_TASK'))
    mem = os.getenv('SLURM_MEM_PER_CPU')
    mem = str(int(mem)*p)+'MB'

    # Initialise Dask cluster and client interface
    initialize(interface = 'ib0', nthreads=p, local_directory='/tmp', memory_limit=mem)
    client = Client()

    # We expect SLURM_NTASKS-2 workers
    N = int(os.getenv('SLURM_NTASKS'))-2 

    # Wait for these workers and report
    client.wait_for_workers(n_workers=N)

    num_workers = len(client.scheduler_info()['workers'])
    print("%d workers available and ready"%num_workers)

    # Code which uses Dask features...

``` 
In the above we make use of various environment variables set by SLURM to tell Dask how much 
RAM is available per worker, and how many CPUs each worker can use. We use the high performance infiniband interconnect `ib0` to communicate between scheduler and workers. Temporary/scratch files will be written
to /tmp on whichever compute node the worker processes execute on.
</details>
A SLURM job script which allocates resources for the Dask cluster would be very similar to an MPI4Py program. We request resources across one or more nodes to run multiple tasks. 

<p class="codeblock-label">dask_mpi.slurm</p>
```bash
#!/bin/bash
#SBATCH --nodes=2
#SBATCH --ntasks-per-node=128
#SBATCH --cpus-per-task=1
#SBATCH --mem-per-cpu=3850
#SBATCH --time=08:00:00

module purge
module load {{site.data.software.defaultgcc}} {{site.data.software.defaultmpi}}
module load {{site.data.software.defaultscipy}}
module load {{site.data.software.defaultdask}}

srun python dask_mpi_init.py
```

In the above we request a single CPU for each task, meaning each Dask worker process will use only a single CPU. We may wish to request multiple CPUs per task if the functions to be executed by the Dask workers release the Python GIL, e.g. compute-intensive NumPy operations 

One disadvantage of this method is that two *tasks* are set aside for the master/client script and the scheduler. If making a SLURM resource request for P CPUs per task then 2P CPUs will not be available to run worker processes but will still be charged as part of the job. For large P this might waste considerable resource budget.

### Launching scheduler and worker processes manually

A more elaborate SLURM job script can be used to gain more control over how the Dask cluster is launched. For example, if the computational work is completely dominated by that allocated to the worker processes we might use the entire SLURM resource allocation for these. The scheduler and master/client script are run outside of SLURM.
Rather than using MPI, communication is via a scheduler file written to the job directory.

Suitable Python code to create a client interface to a Dask cluster launched in this way is below.

<details markdown="block" class="detail">
  <summary>Creating a Dask cluster using a scheduler file <code>dask_file_init.py</code>.</summary>
Minimal Python script which initialises a Dask MPI cluster, creates a client interface to it and reports the number of available workers.

<p class="codeblock-label">dask_file_init.py</p>
```python
import os
from dask.distributed import Client

if __name__ == '__main__':

    # Query SLURM environment per worker task
    p = int(os.getenv('SLURM_CPUS_PER_TASK'))
    mem = os.getenv('SLURM_MEM_PER_CPU')
    mem = str(int(mem)*p)+'MB'

    # We use a schedule file name based on the SLURM job id
    sched_file = os.getenv('SLURM_JOB_ID')+'.sched'
    client = Client(scheduler_file=sched_file, nthreads=p, local_directory='/tmp', memory_limit=mem)

    # We expect SLURM_NTASKS workers
    N = int(os.getenv('SLURM_NTASKS'))

    # Wait for workers and report 
    client.wait_for_workers(n_workers=N)
    num_workers = len(client.scheduler_info()['workers'])
    print("%d workers available and ready"%num_workers)

    # Code which uses Dask features...

``` 
In this case the number of available workers is the number of SLURM tasks.
</details>

The following SLURM job script makes a resource request for 21 worker tasks each using 6 CPUs. This leaves 2 CPUs on the node available to run the scheduler and master/client Python script.

<p class="codeblock-label">dask_file.slurm</p>
```bash
#!/bin/bash
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=21
#SBATCH --cpus-per-task=6
#SBATCH --mem-per-cpu=3850
#SBATCH --time=08:00:00

module purge
module load {{site.data.software.defaultgcc}} {{site.data.software.defaultmpi}}
module load {{site.data.software.defaultscipy}}
module load {{site.data.software.defaultdask}}

# Memory per worker
export MEM=$((${SLURM_CPUS_PER_TASK}*${SLURM_MEM_PER_CPU}))

# Scheduler file - the Python code will need to refer to this
export SCHED_FILE=${SLURM_JOB_ID}.sched

# Launch the scheduler 
dask-scheduler --scheduler-file=$SCHED_FILE  &

# Launch the worker processes using the SLURM-allocated resources
srun  dask-worker --local-directory='/tmp' --no-nanny --nthreads=${SLURM_CPUS_PER_TASK} \
--scheduler-file=$SCHED_FILE  --memory-limit=${MEM}M &

# Launch the master/client script
python dask_file_init.py 

```
## concurrent.futures and Dask

Having initialised a Dask cluster, we can extend our earlier example using [Python concurrent.futures](../../gettingstarted/batchq/singlenode#python-concurrentfutures) to use multiple nodes
of Sulis. This is done by using the `map` function of the Dask client in place of the `map` function
provided by a concurrent.futures executor.

<details markdown="block" class="detail">
  <summary>Example code using Dask client.map <code>dask_futures.py</code>.</summary>
Revisits our earlier example now using `client.map`.

<p class="codeblock-label">dask_futures.py</p>
```python
import os
import sys
from dask_mpi import initialize
from dask.distributed import Client

if len(sys.argv) != 2:
    print("Usage ", sys.argv[0]," <N>")
    sys.exit()
else:
    N = int(sys.argv[1])
    
def f(x):
    return x*x

if __name__ == '__main__':

    # Query SLURM environment per worker task
    p = int(os.getenv('SLURM_CPUS_PER_TASK'))
    mem = os.getenv('SLURM_MEM_PER_CPU')
    mem = str(int(mem)*p)+'MB'

    # Initialise Dask cluster and client interface
    initialize(interface = 'ib0', nthreads=p, local_directory='/tmp', memory_limit=mem)
    client = Client()

    # We expect SLURM_NTASKS-2 workers
    N = int(os.getenv('SLURM_NTASKS'))-2 

    # Wait for these workers and report
    client.wait_for_workers(n_workers=N)

    num_workers = len(client.scheduler_info()['workers'])
    print("%d workers available and ready"%num_workers)

    # Create a list of inputs to the function f
    inputs = range(N)
    
    # Evaluate f for all inputs using a pool of processes
    outputs = client.map(f, inputs)

    print([output.result() for output in outputs])

    client.shutdown()

``` 
</details>

This particular code requires us to pass the number of inputs to evaluate as an argument to the master Python script. A suitable SLURM job script would therefore be the following. This would use the above python script `dask_futures.py` to concurrently evaluate 254 inputs to the function `f(x)` across two nodes of Sulis.

<p class="codeblock-label">dask_futures.slurm</p>
```bash
#!/bin/bash
#SBATCH --nodes=2
#SBATCH --ntasks-per-node=128
#SBATCH --cpus-per-task=1
#SBATCH --mem-per-cpu=3850
#SBATCH --time=08:00:00

module purge
module load {{site.data.software.defaultgcc}} {{site.data.software.defaultmpi}}
module load {{site.data.software.defaultscipy}}
module load {{site.data.software.defaultdask}}

srun python dask_futures.py 254

```

The number of inputs to process needn't match the number of Dask workers available. A common usage would be to work through a number of inputs much greater than the number of workers, i.e. each worker processes multiple inputs. It is desirable to choose a number of workers such that each will process an equal amount of work. This minimises idle workers when the remaining number of inputs to process is less than the number of workers, and hence avoids wasting compute resource.

## Joblib backend

Another simple way to make use of a Dask cluster is to use the backend to Joblib it provides. We can extend our previous [Joblib example](../../gettingstarted/batchq/singlenode#python-joblib) to use the pool of Dask workers to execute calls to a function concurrently.

<details markdown="block" class="detail">
  <summary>Using the Dask backend to Joblib <code>dask_joblib.py</code>.</summary>
Revisits our earlier example now using multiple nodes.

<p class="codeblock-label">dask_joblib.py</p>
```python
import os
import sys
from dask_mpi import initialize
from dask.distributed import Client

import joblib

if len(sys.argv) != 2:
    print("Usage ", sys.argv[0]," <N>")
    sys.exit()
else:
    N = int(sys.argv[1])
    
def f(x):
    return x*x

if __name__ == '__main__':

    # Query SLURM environment per worker task
    p = int(os.getenv('SLURM_CPUS_PER_TASK'))
    mem = os.getenv('SLURM_MEM_PER_CPU')
    mem = str(int(mem)*p)+'MB'

    # Initialise Dask cluster and client interface
    initialize(interface = 'ib0', nthreads=p, local_directory='/tmp', memory_limit=mem)
    client = Client()

    # We expect SLURM_NTASKS-2 workers
    N = int(os.getenv('SLURM_NTASKS'))-2 

    # Wait for these workers and report
    client.wait_for_workers(n_workers=N)

    num_workers = len(client.scheduler_info()['workers'])
    print("%d workers available and ready"%num_workers)

    # Create a list of inputs to the function f
    inputs = range(N)
    
    # Associate a list of outputs with delayed calls to f
    # with p processes available to evaluate them.
    with joblib.parallel_backend('dask'):
        output_list = joblib.Parallel(n_jobs=N)(joblib.delayed(f)(i) for i in inputs)

    # Printing the outputs will cause then to be evaluated
    for output in output_list:
        print(output)

    client.shutdown()

``` 
</details>

As in the previous section, this particular code requires us to pass the number of inputs to evaluate as an argument to the master Python script. A suitable SLURM job script would therefore be the following. This would use the above python script `dask_futures.py` to concurrently evaluate 254 inputs to the function `f(x)` across two nodes of Sulis.

<p class="codeblock-label">dask_joblib.slurm</p>
```bash
#!/bin/bash
#SBATCH --nodes=2
#SBATCH --ntasks-per-node=128
#SBATCH --cpus-per-task=1
#SBATCH --mem-per-cpu=3850
#SBATCH --time=08:00:00

module purge
module load {{site.data.software.defaultgcc}} {{site.data.software.defaultmpi}}
module load {{site.data.software.defaultscipy}}
module load {{site.data.software.defaultdask}}

srun python dask_joblib.py 254
```

Again the number of inputs to process needn't match the number of Dask workers available but it is desirable to make the total amount of work performed by each worker approximately equal to avoid leaving workers idle toward the end of the job.
