---
layout: page
title: GNU parallel
parent: Ensemble jobs
grand_parent: Advanced Topics
nav_order: 2
---

# GNU parallel
{: .no_toc }

1. TOC
{:toc}

GNU parallel is a general purpose command-line tool for launching many concurrent instances of a program to work through a given list of inputs. The number of inputs can be greater than the number of instances.

In the context of a SLURM-managed HPC cluster, parallel can be used to launch multiple concurrent instances of `srun` within a _single_ job. Each instance uses a subset of the resources allocated to the job by SLURM to launch a program (which could itself be either a serial or parallel program). Those resources can span multiple nodes, making parallel valuable for establishing large pools of workers to process many inputs to a program concurrently. 

<details markdown="block" class="detail">
  <summary>Understanding GNU parallel.</summary>

The basic operation of parallel can be understood with the following example.

```shell
{{site.data.terminal.prompt}} module load {{site.data.software.defaultgcc}} {{site.data.software.parallel}} 
{{site.data.terminal.prompt}} parallel -j 1 -N 1 echo ::: Hello world from Sulis
Hello
world
from
Sulis
```

Here we have specified that parallel should use 1 instance (`-j 1`) of the program `echo` to work through the list of arguments following the separator (`:::`). We've also specified that we should pass only 1 entry (`-N 1`) from the list of arguments to echo at a time. Parallel has invoked `echo` 4 times, using one word from the list of arguments each time.

Alternatively:

```shell
{{site.data.terminal.prompt}} parallel -j 1 -N 2 echo ::: Hello world from Sulis
Hello world
from Sulis
```

Now parallel has invoked `echo` twice. First passing the first two arguments from the list, and then the second two. The number of consecutive arguments to pass to each instance of the program is specified by the `-N` parameter.

Finally:

```shell
{{site.data.terminal.prompt}} parallel -j 2 -N 2 echo ::: Hello world from Sulis
Hello world
from Sulis
```

Is is not apparent from the output, but now parallel has invoked two instances of `echo` _at the same time_ (because we specified `-j 2`) and passed the first two words from the list following `:::` as the arguments to the first instance, and the second two words as the arguments to the second instance.
</details>

## Submission script example

The following submission script uses GNU parallel to launch 128 concurrent instances of a serial program `./a.out` which expects a single integer command line argument. 

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

We request 128 tasks per node and a single node. Each instance of the serial program will constitute 1 task.  Parallel is used with the option `-j $SLURM_NTASKS` such that it launches 128 concurrent instances of `srun`. We specify arguments to `srun` itself such that each instance launches a single task using the resources allocated by SLURM. These resources may span multiple nodes - this method can be used to launch batches of serial calculations across hundreds or even thousands of CPUs.

<p class="codeblock-label">parallel.slurm</p>
```bash
#!/bin/bash
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=128
#SBATCH --cpus-per-task=1
#SBATCH --mem-per-cpu=3850
#SBATCH --time=08:00:00

module purge
module load {{site.data.software.defaultgcc}} 
module load {{site.data.software.parallel}}

# Parallel should launch one instances of srun per SLURM task
MY_PARALLEL_OPTS="-N 1 --delay .2 -j $SLURM_NTASKS --joblog parallel-${SLURM_JOBID}.log"

# srun itself should launch 1 instance of our program and not oversubscribe resources
MY_SRUN_OPTS="-N 1 -n 1 --exclusive"

# Use parallel to launch srun with these options
parallel $MY_PARALLEL_OPTS srun $MY_SRUN_OPTS ./a.out ::: {0..1023}
```

The compiled executable `a.out` could be replaced with `python my_script.py` for an equivalent
Python script which take a single integer command line argument.

Note that we've used the shell expansion `{0..1023}` to avoid typing out the integers 
0 to 1023 manually. Note also that this argument list is longer than the number of tasks requested. 
The 128 CPUs we've requested will work through the inputs 128 at a time, running our serial program a total of 1024 times in a _single job script_. 

Additional options to parallel enforce a delay of 0.2 seconds between each invocation of `srun` to 
avoid overloading SLURM with simultaneous resource allocation requests, and specify a log file
we can use to monitor progress. 


## Input handling

Usually our serial program will need more instructive input than a single integer argument. The following examples illustrate two common scenarios for passing more complex inputs into programs launched via parallel.

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
The following example does this by creating a list of input files and passing this
as the list of arguments parallel should pass. Here we launch one instance of the program
per input so that each CPU will only process one instance of our program.

The list of input files is created using the `ls` command as above using backtick substitution.

<p class="codeblock-label">parallel_files.slurm</p>
```bash
#!/bin/bash
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=8
#SBATCH --cpus-per-task=1
#SBATCH --mem-per-cpu=3850
#SBATCH --time=08:00:00

module purge
module load {{site.data.software.defaultgcc}} 
module load {{site.data.software.parallel}}

# Parallel should launch one instance of srun per SLURM task
MY_PARALLEL_OPTS="-N 1 --delay .2 -j $SLURM_NTASKS --joblog parallel-${SLURM_JOBID}.log"

# srun itself should launch 1 instance of our program and not oversubscribe resources
MY_SRUN_OPTS="-N 1 -n 1 --exclusive"

# Use parallel to launch srun with these options
parallel $MY_PARALLEL_OPTS srun $MY_SRUN_OPTS ./a.out ::: `ls -1 *input`
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

We now need to tell parallel that each instance of the program expects 3 arguments, and use the contents of `arglist.txt` as the complete list of arguments to process.

<p class="codeblock-label">parallel_args.slurm</p>
```bash
#!/bin/bash
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=8
#SBATCH --cpus-per-task=1
#SBATCH --mem-per-cpu=3850
#SBATCH --time=08:00:00

module purge
module load {{site.data.software.defaultgcc}} 
module load {{site.data.software.parallel}}

# Parallel should launch one instance of srun per SLURM task with *3* arguments each
MY_PARALLEL_OPTS="-N 3 --delay .2 -j $SLURM_NTASKS --joblog parallel-${SLURM_JOBID}.log"

# srun itself should launch 1 instance of our program and not oversubscribe resources
MY_SRUN_OPTS="-N 1 -n 1 --exclusive"

# Use parallel to launch srun with these options
parallel $MY_PARALLEL_OPTS srun $MY_SRUN_OPTS ./a.out ::: `cat arglist.txt`
```



## Using a bash function for pre/post processing

One particular advantage of GNU parallel is that the command launched needn't be an executable. It can be a bash function. This function might be used to (for example) create a new directory in which to run each instance of the program and/or delete any temporary/unneeded files it creates.

As an example, consider a program `my_prog` which reads the name of an input file as its only command line argument. The program generates various temporary files in its working directory, but we are only interesting in keeping the main output file. The input files we wish to process are kept in the directory `inputs` and we wish to copy the output file to `outputs`.

The following SLURM job script uses parallel to launch a Bash function which accomplishes this.

<p class="codeblock-label">bash_function.slurm</p>
```bash
#!/bin/bash
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=128
#SBATCH --cpus-per-task=1
#SBATCH --mem-per-cpu=3850
#SBATCH --time=08:00:00

module purge
module load {{site.data.software.defaultgcc}} 
module load {{site.data.software.parallel}}

############################################
# Function to execute for every input file #
############################################
function run_calc()
{
    inpfile=$1 # input file to process

    # Make a temporary directory and change into it
    tmpdir=`mktemp -d -p ./`
    cd $tmpdir

    # Run the program with this input file. Launch
    # using srun to use 1 of the SLURM allocated tasks.
    srun -N 1 -n 1 --exclusive my_prog $inpfile

    # Copy output file to outputs
    cd ../
    cp ${tmpdir}/*output outputs/

    # Delete the temporary directory
    rm -rf $tmpdir

}

export -f run_calc

# Parallel should launch one instance of srun per SLURM task
MY_PARALLEL_OPTS="-N 1 --delay .2 -j $SLURM_NTASKS --joblog parallel-${SLURM_JOBID}.log"

# Use parallel to launch srun with these options
parallel $MY_PARALLEL_OPTS run_calc ::: `ls -1 inputs/*input`
```

This mechanism can be used to implement a range of pre- or post-processing of data around
execution of a program.  


## Using parallel to launch parallel programs

Use of parallel is not restricted to serial jobs. It may be desirable to implement a workflow in which a large number of nodes is used to run many concurrent instances of an MPI or otherwise parallel program. In these cases the resource request should reflect the total number of tasks to be launched, so an MPI program which uses 8 tasks and 2 CPUs per task should request 64 tasks per node as in the following example.

<p class="codeblock-label">parallel.slurm</p>
```bash
#!/bin/bash
#SBATCH --nodes=4
#SBATCH --ntasks-per-node=64
#SBATCH --cpus-per-task=2
#SBATCH --mem-per-cpu=3850
#SBATCH --time=08:00:00

module purge
module load {{site.data.software.defaultgcc}} 
module load {{site.data.software.parallel}}

# Set number of MPI tasks to use per instance of my_mpi_prog
TASKS_PER_PROG=8

# Set OMP_NUM_THREADS
export OMP_NUM_THREADS=$SLURM_CPUS_PER_TASK

# Calculate how many concurrent MPI instances to use
NJOBS=$((${SLURM_NTASKS}/${TASKS_PER_PROG}))

# Parallel should launch NJOBS instances of srun at any one time
MY_PARALLEL_OPTS="-N 1 --delay .2 -j ${NJOBS} --joblog parallel-${SLURM_JOBID}.log"

# Each invocation of srun should launch $TASKS_PER_PROG tasks
MY_SRUN_OPTS="-N 1 -n ${TASKS_PER_PROG} -c ${SLURM_CPUS_PER_TASK} --exclusive"

# Use parallel to launch srun with these options
parallel $MY_PARALLEL_OPTS srun $MY_SRUN_OPTS my_mpi_prog ::: {0..1023}
```
This would launch 256 concurrent instances of `my_mpi_prog` as an 8-task MPI program with each task using 2 CPUs per task. The instances would work through the 1024 inputs 256 at a time.

Here we've been careful to ensure that none of these 256 instances will be split across multiple nodes as 128 is divisible by 16. This would normally be optimal in terms of performance.

