---
layout: page
title: Job arrays 
parent: Ensemble jobs
grand_parent: Advanced Topics
nav_order: 1
---

# Job arrays
{: .no_toc }

1. TOC
{:toc}

SLURM provides functionality for _array jobs_. These are collections of jobs which differ only in the
input passed to the program launched with `srun`. A single job script is prepared and submitted with `sbatch`. This results in a number of jobs being added to the queue as independent instances of the job script. Each instance is assigned a unique task ID. The job script can use this ID to determine which inputs to pass into the program launched by the script.

## Array submission script

Use of job arrays can be demonstrated with a simple program which reports its command line argument. An equivalent Python script using `sys.argv` could substitute.

<details markdown="block" class="detail">
  <summary>C program which prints its command line arguments <code>c_arg.c</code>.</summary>
Trivial C program to demonstrate use of job arrays in SLURM.

<p class="codeblock-label">c_arg.c</p>
```c
#include <stdio.h>
#include <stdio.h>
#include <stdlib.h>

int main(int argc, char* argv[]) {

  int i;
  printf("Arguments : ");
  for (i=1;i<argc;i++) {
    printf("%s ",argv[i]);
  }
  printf("\n");

  exit(EXIT_SUCCESS);
}

``` 
This can be compiled to an executable `a.out` via:

```shell
{{site.data.terminal.prompt}} module load {{site.data.software.defaultgcc}} 
{{site.data.terminal.prompt}} gcc c_arg.c
```
</details>

The resource request section of an array job script should request the resources needed for a single
element of the job array. In this case each element of the job array will launch a single serial 
program, so we request 1 node, 1 task per node and one CPU per task.

The additional request `#SBATCH --array=0-31` indicates that we require 32 instances of this script to be submitted, indexed by the integers 0 to 31. The index for the current instance, available via the envronment variable `SLURM_ARRAY_TASK_ID` is passed to our compiled executable as a command line argument when launched with `srun`.

<p class="codeblock-label">jobarray.slurm</p>
```bash
#!/bin/bash
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=1
#SBATCH --mem-per-cpu=3850
#SBATCH --time=08:00:00
#SBATCH --account=suxxx-somebudget
#SBATCH --array=0-31

module purge
module load {{site.data.software.defaultgcc}} 

srun ./a.out $SLURM_ARRAY_TASK_ID
```
Job array scripts are submitted as any other.

```shell
{{site.data.terminal.prompt}} sbatch jobarray.slurm
Submitted batch job 207566

```

The individual elements of the array will appear in the queue as separate jobs, identified by the SLURM job number followed by an underscore and the index of each instance.

```shell
{{site.data.terminal.prompt}} squeue
             JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
          207566_0   compute jobarray   phseal CG       0:04      1 node046
          207566_1   compute jobarray   phseal CG       0:06      1 node046
          207566_2   compute jobarray   phseal CG       0:10      1 node046
          207566_3   compute jobarray   phseal CG       0:12      1 node046
          207566_4   compute jobarray   phseal CG       0:08      1 node046
          207566_5   compute jobarray   phseal CG       0:13      1 node046
          207566_6   compute jobarray   phseal  R       0:14      1 node046
          207566_7   compute jobarray   phseal  R       0:14      1 node046
          207566_8   compute jobarray   phseal  R       0:14      1 node046
          207566_9   compute jobarray   phseal  R       0:14      1 node046
         207566_10   compute jobarray   phseal  R       0:14      1 node046
         207566_11   compute jobarray   phseal  R       0:14      1 node046
         207566_12   compute jobarray   phseal  R       0:14      1 node046
         207566_13   compute jobarray   phseal  R       0:14      1 node046
         207566_14   compute jobarray   phseal  R       0:14      1 node046
         207566_15   compute jobarray   phseal  R       0:14      1 node046
         207566_16   compute jobarray   phseal  R       0:14      1 node046
         207566_17   compute jobarray   phseal  R       0:14      1 node046
         207566_18   compute jobarray   phseal  R       0:14      1 node046
         207566_19   compute jobarray   phseal  R       0:14      1 node046
         207566_20   compute jobarray   phseal  R       0:14      1 node046
         207566_21   compute jobarray   phseal  R       0:14      1 node046
         207566_22   compute jobarray   phseal  R       0:14      1 node046
         207566_23   compute jobarray   phseal  R       0:14      1 node046
         207566_24   compute jobarray   phseal  R       0:14      1 node046
         207566_25   compute jobarray   phseal  R       0:14      1 node046
         207566_26   compute jobarray   phseal  R       0:14      1 node046
         207566_27   compute jobarray   phseal  R       0:14      1 node046
         207566_28   compute jobarray   phseal  R       0:14      1 node046
         207566_29   compute jobarray   phseal  R       0:14      1 node046
         207566_30   compute jobarray   phseal  R       0:14      1 node046
         207566_31   compute jobarray   phseal  R       0:14      1 node046
```

Similarly the output of each element will be written to `slurm-xxxxxx_yy.out` where xxxxxx is the SLURM job number and yy is in the index of the instance writing output to that file. In this case our simple program will write the arguments it was given and quit.

```shell
{{site.data.terminal.prompt}} cat slurm-207566_*.out
Arguments : 0 
Arguments : 1 
Arguments : 10 
Arguments : 11 
Arguments : 12 
Arguments : 13 
Arguments : 14 
Arguments : 15 
Arguments : 16 
Arguments : 17 
Arguments : 18 
Arguments : 19 
Arguments : 2 
Arguments : 20 
Arguments : 21 
Arguments : 22 
Arguments : 23 
Arguments : 25 
Arguments : 26 
Arguments : 28 
Arguments : 29 
Arguments : 3 
Arguments : 4 
Arguments : 5 
Arguments : 6 
Arguments : 7 
Arguments : 8 
Arguments : 9 
```
Each instance of the script has passed a different argument to `a.out` as desired.


## Input handling

Usually the array job script will need to perform some processing on `SLURM_ARRAY_TASK_ID` to produce inputs for the program launched by `srun`. Two methods for accomplishing this are demonstrated below.

### Processing a list of input files

Many scientific software packages expect a single command line argument which specifies an input file
which should be read by the program. Consider creating a job script which launches one instance for a program for each of the 8 input files in a directory.

```shell
{{site.data.terminal.prompt}} ls -1 *input
t1.1p5.0.input
t1.1p6.0.input
t1.2p5.0.input
t1.2p6.0.input
t1.3p5.0.input
t1.3p6.0.input
t1.4p5.0.input
t1.4p6.0.input
```
The following example does this by creating a bash array from this list of input files and then uses 
`SLURM_ARRAY_TASK_ID` to specify which entry in that array should be used by the current
instance of the job script. The number of elements in the array should match the number
of input files.


<p class="codeblock-label">arrayinputs.slurm</p>
```bash
#!/bin/bash
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=1
#SBATCH --mem-per-cpu=3850
#SBATCH --time=08:00:00
#SBATCH --account=suxxx-somebudget
#SBATCH --array=0-7

module purge
module load {{site.data.software.defaultgcc}} 

inputfiles=(*.input) # Bash array of input files
srun ./a.out ${inputfiles[${SLURM_ARRAY_TASK_ID}]}
```

### Processing a list of command line arguments

Other software will expect a number of command line arguments rather than (or as well as)
an input file. For example our `a.out` executable might expect to be given arguments such as the following.
```bash
./a.out --temperature=1.1 --pressure=5.0 --run_length=10000
```
One way to handle this is to create a file which lists the sets of arguments to be used in the array. For example.

<p class="codeblock-label">arglist.txt</p>
```
--temperature=1.1 --pressure=5.0 --run_length=10000
--temperature=1.2 --pressure=5.0 --run_length=10000
--temperature=1.3 --pressure=5.0 --run_length=10000
--temperature=1.4 --pressure=5.0 --run_length=10000
--temperature=1.1 --pressure=6.0 --run_length=10000
--temperature=1.2 --pressure=6.0 --run_length=10000
--temperature=1.3 --pressure=6.0 --run_length=10000
--temperature=1.4 --pressure=6.0 --run_length=10000
```

The following is a job array script which passes one of the lines in this file as the arguments to each instance of the program.

<p class="codeblock-label">argfile.slurm</p>
```bash
#!/bin/bash
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=1
#SBATCH --mem-per-cpu=3850
#SBATCH --time=08:00:00
#SBATCH --account=suxxx-somebudget
#SBATCH --array=0-7

module purge
module load {{site.data.software.defaultgcc}} 

# Array inputs holds one entry per line in arglist.txt
IFS=$'\n' read -a inputs -d '' <<< `cat arglist.txt`

srun ./a.out ${inputs[$SLURM_ARRAY_TASK_ID]}
```

Care must be taken to ensure that the number of elements in the job array is equal to the number
of entries in arglist.txt. 

## Parallelism within array elements

Each element of a job array can launch a parallel program, using multiple tasks, CPUs or a
hybrid combination of the two. Jobs scripts should be prepared such that the resource request
is a appropriate to a single parallel instance of the program. 

To illustrate, the example above could be modified to launch an MPI program with each of 8 tasks using 2 threads each.
<p class="codeblock-label">argfile_hybrid.slurm</p>
```bash
#!/bin/bash
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=8
#SBATCH --cpus-per-task=2
#SBATCH --mem-per-cpu=3850
#SBATCH --time=08:00:00
#SBATCH --account=suxxx-somebudget
#SBATCH --array=0-7

module purge
module load {{site.data.software.defaultfoss}} 

# Array inputs holds one entry per line in arglist.txt
IFS=$'\n' read -a inputs -d '' <<< `cat arglist.txt`

export OMP_NUM_THREADS=$SLURM_CPUS_PER_TASK
srun ./my_mpi_prog ${inputs[$SLURM_ARRAY_TASK_ID]}
```
This would result in 8 elements of the job array, each using 16 CPUs.

## Disadvantages

Job arrays have some disadvantages over other methods of orchestrating ensembles of small calculations on a HPC system.

1. Each element in the job array processes only a single input when used as above. SLURM allocates resource which might only be used for a short time, incurring overhead. It is preferable for SLURM to allocate resources which are then used to process multiple inputs as per other methods.

2. Although SLURM attempts to pack the elements of the array onto as few nodes as possible, job arrays can still fragment the resource available to workloads which require whole nodes, reducing overall utilisation of the cluster.

3. Running many thousands of array job elements per day can (and does) overload the SAFE accounting system.
                                      
4. In some cases is it desirable for all members of the ensemble to be executing at the same time, which is not guaranteed by job arrays.

Job arrays should ideally only be used in scenarios where a few dozen (up to 256) long-running elements are required. For other workloads which process many hundreds or thousands of inputs that run for a few minutes/hours, [GNU parallel](gnuparallel) should be used as an alternative.
