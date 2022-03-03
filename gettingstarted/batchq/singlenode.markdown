---
layout: page
title: Single node jobs 
parent: Job submission
grand_parent: Getting Started
nav_order: 3
---

# Single node jobs
{: .no_toc }

1. TOC
{:toc}

This page contains example SLURM job scripts which implement some (or no) parallelism via mechanisms which restrict execution to a single compute node. 

Job scripts are text files typically containing the following:

- a shell interpreter, typically `#!/bin/bash`
- a resource request, i.e. the number of nodes, tasks per node, and memory per task as well as the amount of wall clock time required for the job to complete
- any shell environment variables
- any [environment modules](../../gettingstarted/software/modules) to be loaded
- a command to launch the program(s) which will run on the requested resource

<!-- DQ we might need a node here on charging model and node exclusivity. Will we always allocate (and charge for) a whole node to jobs which only use a fraction? --->

## Serial jobs

**IMPORTANT**: Regular submission of serial jobs is strongly discouraged. We do however welcome (and encourage) workflows which implement task-based parallelism by launching many instances of a serial program as a single job. See the [Ensemble jobs](../../advanced/ensemble/) section of these pages for further information. 

<details markdown="block" class="detail">
  <summary>An example serial program in C.</summary>
A trivial serial job can be illustrated with the famous "Hello world" example in C. 

<p class="codeblock-label">hello_world.c</p>
```c 
#include <stdio.h>
int main() {
   printf("Hello World!");
   return 0;
}
``` 
This might be compiled into the executable `a.out` via:
```bash
{{site.data.terminal.prompt}} module load {{site.data.software.defaultgcc}}
{{site.data.terminal.prompt}} gcc hello_world.c
```
</details>

A job script suitable for an entirely serial calculation should request one node, a single task on that node and a single CPU per task. This is accomplished via special comments (starting with #).

<p class="codeblock-label">serial.slurm</p>
```bash
#!/bin/bash
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=1
#SBATCH --mem-per-cpu={{site.data.slurm.cnode_ram_per_core}}
#SBATCH --time=08:00:00
#SBATCH --account=suxxx-somebudget

module purge
module load {{site.data.software.defaultgcc}}

srun ./a.out
```
In the above example we request {{site.data.slurm.cnode_ram_per_core}} MB of RAM, which is the total amount of RAM available to user jobs in compute node, divided by the number of CPUs in the node. This resource is requested for 8 hours (hh:mm:ss) of walltime.

The script then loads modules to reproduce the environment in which the serial executable `a.out` was compiled. 

The SLURM command `srun` is then used to launch the executable using the resources requested. In this case it will launch a single instance (task) of the program using a single CPU. No other arguments to `srun` are necessary in this case, it is aware of the resources assigned by SLURM.

The above assumes the executable file `a.out` already exists, and is located in the working directory from which the `sbatch` submission command was executed. The above example cat be submitted with:
```shell
{{site.data.terminal.prompt}} sbatch serial.slurm
Submitted batch job 212677
```
Anything written to stdout by the program, i.e. with printf (C) or write (Fortran) will be captured 
by SLURM and written to the file `slurm-xxxx.out` where `xxxx` is the unique ID of the job (212677 in the example above). 

<details markdown="block" class="detail">
  <summary>Output from example program.</summary>
```shell
{{site.data.terminal.prompt}} cat slurm-212677.out
Hello World!
```
</details>

### Note on serial jobs with large memory requirements

In some cases it may be necessary to request more RAM for serial jobs,  e.g. for in-memory post processing of data. In such cases scripts should request multiple ``cpus-per-task`` to access more memory, leaving the additional CPUs unused.

Sulis does contain {{site.data.slurm.fatnode_partition_size}} high memory nodes with {{site.data.slurm.fatnode_ram_per_core}} MB of RAM available per CPU. These are available for memory-intensive processing on request.

## OpenMP jobs

Jobs which consist of a single task that uses multiple CPUs via threaded parallelism (usually implemented in OpenMP) can use upto {{site.data.slurm.cnode_cores_per_node}} CPUs per job.

<details markdown="block" class="detail">
  <summary>An example OpenMP program in C.</summary>
An extension of our trivial example from above. 

<p class="codeblock-label">omp_hello.c</p>
```c 
#include <stdio.h>
#include <omp.h>
int main ()  {
  int nthreads, tid;
#pragma omp parallel private(tid)
  {
    tid = omp_get_thread_num();
    printf("Hello world from thread = %d\n", tid);
    if (tid == 0) {
        nthreads = omp_get_num_threads();
        printf("Number of threads = %d\n", nthreads);
    }
  }
}
``` 
This might be compiled into the executable `a.out` via:
```bash
{{site.data.terminal.prompt}} module load {{site.data.software.defaultgcc}}
{{site.data.terminal.prompt}} gcc -fopenmp omp_hello.c
```
</details>

A job script suitable for a pure OpenMP program should request a single node, specify a single task and as many CPUs per task as the program can usefully use. 

<p class="codeblock-label">openmp.slurm</p>
```bash
#!/bin/bash
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task={{site.data.slurm.cnode_cores_per_node}}
#SBATCH --mem-per-cpu={{site.data.slurm.cnode_ram_per_core}}
#SBATCH --time=08:00:00
#SBATCH --account=suxxx-somebudget

module purge
module load {{site.data.software.defaultgcc}}

export OMP_NUM_THREADS=$SLURM_CPUS_PER_TASK

srun ./a.out
```

As with the serial example there is no need to specify additional arguments to `srun`. However we do manually set the `OMP_NUM_THREADS` environment variable to ensure our OpenMP program launches only as many threads as SLURM expects. 

Submission proceeds as per any other job script.

```shell
{{site.data.terminal.prompt}} sbatch openmp.slurm
Submitted batch job 212678
```

Note that very few OpenMP codes will effectively use all {{site.data.slurm.cnode_cores_per_node}} core available on a Sulis compute node. It will likely be more efficient to run multiple instances per node using fewer threads each. Users should experiment to find optimal throughput.

<details markdown="block" class="detail">
  <summary>Output from example program.</summary>
```shell
{{site.data.terminal.prompt}} cat slurm-212678.out
Hello world from thread = 6
Hello world from thread = 26
Hello world from thread = 32
Hello world from thread = 35
Hello world from thread = 42
Hello world from thread = 1
Hello world from thread = 2
Hello world from thread = 47
Hello world from thread = 45
Hello world from thread = 46
Hello world from thread = 4
Hello world from thread = 50
Hello world from thread = 51
Hello world from thread = 54
Hello world from thread = 7
Hello world from thread = 58
Hello world from thread = 59
Hello world from thread = 60
Hello world from thread = 61
Hello world from thread = 62
Hello world from thread = 63
Hello world from thread = 9
Hello world from thread = 66
Hello world from thread = 67
Hello world from thread = 69
Hello world from thread = 70
Hello world from thread = 72
Hello world from thread = 73
Hello world from thread = 74
Hello world from thread = 76
Hello world from thread = 79
Hello world from thread = 80
Hello world from thread = 82
Hello world from thread = 83
Hello world from thread = 84
Hello world from thread = 86
Hello world from thread = 87
Hello world from thread = 90
Hello world from thread = 89
Hello world from thread = 92
Hello world from thread = 94
Hello world from thread = 95
Hello world from thread = 98
Hello world from thread = 15
Hello world from thread = 100
Hello world from thread = 102
Hello world from thread = 16
Hello world from thread = 106
Hello world from thread = 108
Hello world from thread = 109
Hello world from thread = 110
Hello world from thread = 17
Hello world from thread = 113
Hello world from thread = 115
Hello world from thread = 117
Hello world from thread = 118
Hello world from thread = 120
Hello world from thread = 121
Hello world from thread = 123
Hello world from thread = 125
Hello world from thread = 126
Hello world from thread = 0
Hello world from thread = 25
Hello world from thread = 28
Hello world from thread = 30
Hello world from thread = 33
Hello world from thread = 38
Hello world from thread = 39
Hello world from thread = 5
Hello world from thread = 44
Hello world from thread = 48
Hello world from thread = 52
Hello world from thread = 55
Hello world from thread = 57
Hello world from thread = 65
Hello world from thread = 71
Hello world from thread = 75
Hello world from thread = 78
Hello world from thread = 14
Hello world from thread = 85
Hello world from thread = 88
Hello world from thread = 91
Hello world from thread = 96
Hello world from thread = 97
Hello world from thread = 101
Hello world from thread = 104
Hello world from thread = 18
Hello world from thread = 111
Hello world from thread = 114
Hello world from thread = 116
Hello world from thread = 119
Hello world from thread = 21
Hello world from thread = 24
Hello world from thread = 27
Hello world from thread = 29
Hello world from thread = 34
Hello world from thread = 36
Hello world from thread = 40
Hello world from thread = 41
Hello world from thread = 3
Hello world from thread = 43
Hello world from thread = 49
Hello world from thread = 53
Hello world from thread = 56
Hello world from thread = 64
Hello world from thread = 10
Hello world from thread = 68
Hello world from thread = 11
Hello world from thread = 77
Hello world from thread = 81
Hello world from thread = 127
Hello world from thread = 13
Hello world from thread = 93
Hello world from thread = 8
Hello world from thread = 99
Hello world from thread = 103
Hello world from thread = 105
Hello world from thread = 107
Hello world from thread = 112
Hello world from thread = 19
Hello world from thread = 22
Hello world from thread = 20
Hello world from thread = 122
Hello world from thread = 124
Hello world from thread = 23
Number of threads = 128
Hello world from thread = 31
Hello world from thread = 37
Hello world from thread = 12
```
</details>

## Python multiprocessing

Python programmers may with to use the multprocessing package to implement parallelism within a single node. SLURM treads a Python program which spawns or forks multiple sub-processes as a single task which uses a "pool" of multiple CPUs.

<details markdown="block" class="detail">
  <summary>An example Python multiprocessing code <code>example_mp.py</code>.</summary>
This squares the first `N` integers, distributing the work over a pool of `p` processes. 

<p class="codeblock-label">example_mp.py</p>
```python
import sys
from multiprocessing import Pool

if len(sys.argv) != 3:
    print("Usage ", sys.argv[0]," <p> <N>")
    sys.exit()
else:
    p = int(sys.argv[1])
    N = int(sys.argv[2])
    
def f(x):
    return x*x

if __name__ == '__main__':

    # Create a list of inputs to the function f
    inputs = range(N)
    
    # Evaluate f for all inputs using a pool of processes
    with Pool(p) as my_pool:
        print(my_pool.map(f, inputs))
```
</details>

The following job script runs this example. The number of CPUs per task allocated by SLURM is passed into the Python script as the first argument and used to set the size of the multiprocessing pool equal to the number of CPUs per task allocated by SLURM. 

Note that the number of function inputs (specified by the second argument) does not need to match the size of the pool. Optimal load balancing across processors will occur when the number of inputs is a multiple of the pool size, assuming each input requires a similar amount of CPU time.

<p class="codeblock-label">multiprocessing.slurm</p>
```bash
#!/bin/bash
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task={{site.data.slurm.cnode_cores_per_node}}
#SBATCH --mem-per-cpu={{site.data.slurm.cnode_ram_per_core}}
#SBATCH --time=08:00:00
#SBATCH --account=suxxx-somebudget

module purge
module load {{site.data.software.defaultgcccore}} {{site.data.software.defaultpython}}

export p=$SLURM_CPUS_PER_TASK # Size of multiprocessing pool
export N=128                  # Number of inputs

# example_mp.py sets pool size from the first argument
srun python example_mp.py $p $N
```

<!-- A note of caution on use of the [Python subprocesses](https://docs.python.org/3/library/subprocess.html) within functions invoked by a multiprocessing pool. This may be desirable if using Python to launch multiple instances of a compiled serial program to implement an ensemble computing workflow. However the additional processes may be starved of CPU resource resulting in poor performance without specifying appropriate options to `srun`. This is discussed in more detail in the [Advanced topics](../../advanced/ensemble/subprocess/) section. -->

Submission proceeds as per any other job script.

```shell
{{site.data.terminal.prompt}} sbatch multiprocessing.slurm
Submitted batch job 212679
```
<details markdown="block" class="detail">
  <summary>Output from example program.</summary>
```shell
{{site.data.terminal.prompt}} cat slurm-212679.out
[0, 1, 4, 9, 16, 25, 36, 49, 64, 81, 100, 121, 144, 169, 196, 225, 256, 289, 324, 361, 400, 441, 484, 529, 576, 625, 676, 729, 784, 841, 900, 961, 1024, 1089, 1156, 1225, 1296, 1369, 1444, 1521, 1600, 1681, 1764, 1849, 1936, 2025, 2116, 2209, 2304, 2401, 2500, 2601, 2704, 2809, 2916, 3025, 3136, 3249, 3364, 3481, 3600, 3721, 3844, 3969, 4096, 4225, 4356, 4489, 4624, 4761, 4900, 5041, 5184, 5329, 5476, 5625, 5776, 5929, 6084, 6241, 6400, 6561, 6724, 6889, 7056, 7225, 7396, 7569, 7744, 7921, 8100, 8281, 8464, 8649, 8836, 9025, 9216, 9409, 9604, 9801, 10000, 10201, 10404, 10609, 10816, 11025, 11236, 11449, 11664, 11881, 12100, 12321, 12544, 12769, 12996, 13225, 13456, 13689, 13924, 14161, 14400, 14641, 14884, 15129, 15376, 15625, 15876, 16129]
```
Note that the list of outputs is ordered as per the list of inputs.
</details>

## Python concurrent.futures

Similar to multiprocessing, the [concurrent.futures](https://docs.python.org/3/library/concurrent.futures.html) module introduced in Python 3.2 provides a means to distribute work over a pool of either threads or processes. 

Use of threads for parallelism in Python has a number of pitfalls due to the global interpreter lock or GIL. This prevents multiple threads from executing code at the same time, limiting their utility for parallelism. Launching threads generally incurs less overhead than launching processes, so if the work to be executed by the worker pool releases the GIL this _can_ be a more efficient option. Here we focus on the use of processes. 

A particular advantage of using [concurrent.futures](https://docs.python.org/3/library/concurrent.futures.html) is that the resulting code requires only minimal modification to take advantage of a worker pool distributed over many nodes via [mpi4py.futures](https://mpi4py.readthedocs.io/en/stable/mpi4py.futures.html). See the [MPI section](mpi) section of this documentation for more information.

<details markdown="block" class="detail">
  <summary>An example Python code using concurrent.futures<code>example_futures.py</code>.</summary>
This squares the first `N` integers, distributing the work over a pool of `p` processes. 

<p class="codeblock-label">example_futures.py</p>
```python
import sys
import concurrent.futures

if len(sys.argv) != 3:
    print("Usage ", sys.argv[0]," <p> <N>")
    sys.exit()
else:
    p = int(sys.argv[1])
    N = int(sys.argv[2])
    
def f(x):
    return x*x

if __name__ == '__main__':

    # Create a list of inputs to the function f
    inputs = range(N)
    
    # Evaluate f for all inputs using a pool of processes
    with concurrent.futures.ProcessPoolExecutor(max_workers=p) as executor:
        results = executor.map(f, inputs)

    print([result for result in results])
```
</details>

The SLURM job script for a Python script which uses concurrent.futures is very similar to that for multiprocessing. The script uses `srun` to launch a single task which uses multiple CPUs to establish the worker pool.

<p class="codeblock-label">multiprocessing.slurm</p>
```bash
#!/bin/bash
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=128
#SBATCH --mem-per-cpu=3850
#SBATCH --time=08:00:00
#SBATCH --account=suxxx-somebudget

module purge
module load {{site.data.software.defaultgcccore}} {{site.data.software.defaultpython}}

export p=$SLURM_CPUS_PER_TASK # Size of multiprocessing pool
export N=128                  # Number of inputs

# example_mp.py sets pool size from the first argument
srun python example_futures.py $p $N
```

This should match the output of the above multiprocessing example exactly.

As with multiprocessing the number of inputs to process can be larger than the size of the worker pool. Assuming each input takes a similar length of time to process, optimal utilisation will involve a number of inputs equal to an integer multiple of the worker pool size.


## Python joblib

Joblib is an alternative method of evaluating functions for a list of inputs in Python with the work distributed over multiple CPUs in a node. It is included as part of the SciPy-bundle environment module. 

A particular advantage of joblib over multiprocessing is that it can be easily adapted to implement parallelism over multiple nodes in a cluster by using the [Dask](https://dask.org/) backend as discussed in the [Advanced topics](../../advanced/ensemble/joblib/) section. For now we will restrict ourselves to the standard backend which is restricted to parallelism over a single node.

<details markdown="block" class="detail">
  <summary>An example Python joblib code <code>example_joblib.py</code>.</summary>
This squares the first `N` integers, distributing the work over a pool of `p` processes. 

<p class="codeblock-label">example_joblib.py</p>
```python
import sys
from joblib import Parallel, delayed

if len(sys.argv) != 3:
    print("Usage ", sys.argv[0]," <p> <N>")
    sys.exit()
else:
    p = int(sys.argv[1])
    N = int(sys.argv[2])
    
def f(x):
    return x*x

if __name__ == '__main__':

    # Create a list of inputs to the function f
    inputs = range(N)
    
    # Associate a list of outputs with delayed calls to f
    # with p processes available to evaluate them.
    outputs = Parallel(n_jobs=p)(delayed(f)(i) for i in inputs)

    # Printing the outputs will cause then to be evaluated
    print(outputs)
```
</details>

The required job script is nearly identical to the multiprocessing example above with
the addition of the SciPy-bundle module. As before, the number of processes to use is passed into the python script as an argument.

<p class="codeblock-label">joblib.slurm</p>
```bash
#!/bin/bash
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task={{site.data.slurm.cnode_cores_per_node}}
#SBATCH --mem-per-cpu={{site.data.slurm.cnode_ram_per_core}}
#SBATCH --time=08:00:00
#SBATCH --account=suxxx-somebudget

module purge
module load {{site.data.software.defaultfoss}} {{site.data.software.defaultscipy}}

export p=$SLURM_CPUS_PER_TASK # Value to use as n_jobs for joblib
export N=128                  # Number of inputs

# example_joblib.py sets n_jobs from the first argument
srun python example_joblib.py $p $N
```

<!-- Similar caveats apply if using subprocess within junctions evaluated in parallel via joblib. This is discussed in more detail in the [Advanced topics](../../advanced/ensemble/subprocess/) section. -->

Submission proceeds as per any other job script.

```shell
{{site.data.terminal.prompt}} sbatch joblib.slurm
Submitted batch job 212680
```

<details markdown="block" class="detail">
  <summary>Output from example program.</summary>
```shell
{{site.data.terminal.prompt}} cat slurm-212680.out
[0, 1, 4, 9, 16, 25, 36, 49, 64, 81, 100, 121, 144, 169, 196, 225, 256, 289, 324, 361, 400, 441, 484, 529, 576, 625, 676, 729, 784, 841, 900, 961, 1024, 1089, 1156, 1225, 1296, 1369, 1444, 1521, 1600, 1681, 1764, 1849, 1936, 2025, 2116, 2209, 2304, 2401, 2500, 2601, 2704, 2809, 2916, 3025, 3136, 3249, 3364, 3481, 3600, 3721, 3844, 3969, 4096, 4225, 4356, 4489, 4624, 4761, 4900, 5041, 5184, 5329, 5476, 5625, 5776, 5929, 6084, 6241, 6400, 6561, 6724, 6889, 7056, 7225, 7396, 7569, 7744, 7921, 8100, 8281, 8464, 8649, 8836, 9025, 9216, 9409, 9604, 9801, 10000, 10201, 10404, 10609, 10816, 11025, 11236, 11449, 11664, 11881, 12100, 12321, 12544, 12769, 12996, 13225, 13456, 13689, 13924, 14161, 14400, 14641, 14884, 15129, 15376, 15625, 15876, 16129]
```
Note that the list of outputs is ordered as per the list of inputs.
</details>

## Parallel package in R

R scripts may use the `parallel` package to implement calcualtions that use multiple cores within a node, either explicitly or as part of functions within other packages. An example R script which uses `mcapply` to parallelise a calculation is below.

<details markdown="block" class="detail">
  <summary>An example R code using the `parallel` package <code>example.R</code>.</summary>
This generates N samples from the standard normal distribution, and then performs a bootstrap analsys of the mean by resampling (with replacement) k times from these N samples. The distribution of means is compared to the standard error of the original sample set and the distribution of means is compared to the expected form.

The input parameters N and k are read as command line arguments.

<p class="codeblock-label">example.R</p>
```R
#!/usr/bin/env Rscript

# Get command line arguments N and k
args <- commandArgs(trailingOnly=TRUE)
N <- strtoi(args[1])
k <- strtoi(args[2])

# Generate N samples from the normal distribution, then do a 
# bootstrap error analysis on the mean with k trials
samples <- rnorm(N)

resample <- function(trial) {
  new_samples <- sample(samples, N, replace=TRUE)
  return(mean(new_samples))
}

# Load the parallel library
library(parallel)

# Conduct k trials in parallel
timing =system.time({
  resampled_means <- unlist(mclapply(1:k, resample))
})

# Print timing
print(timing)

av <- mean(samples)        # mean
se <- sd(samples)/sqrt(N)  # standard error

# Histogram of the k resampled means and expected distribution
hist(resampled_means, prob = TRUE)
curve(dnorm(x, av, se), col = "red", add = TRUE)
```
</details>

A SLURM job script suitable for running this example on 4 cores is given below. Note that we set the environment variable `MC_CORES` to match the number of CPUs allocated to our (single) task. This controls the number of cores used by the `parallel` package in our call to `mclapply`. We set N and k as shell variables and then pass these into our R script as command line arguments.

<p class="codeblock-label">Rparallel.slurm</p>
```bash
#!/bin/bash
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=4
#SBATCH --mem-per-cpu=3850
#SBATCH --time=00:01:00
#SBATCH --account=suxxx-somebudget

module purge
module load GCC/11.2.0 OpenMPI/4.1.1 R/4.1.2

# Set number of cores used by parallel package
export MC_CORES=$SLURM_CPUS_PER_TASK

# Run script. N and k are arguments
N=10000
k=50000
srun R CMD BATCH "--args $N $k" example.R
```

This trival example runs approximately 4 faster using 4 cores (7.6s) than using 1 (28.7s). Distributing more computationally intensive function calls with `mclapply` should scale to larger core counts than this trivial example.