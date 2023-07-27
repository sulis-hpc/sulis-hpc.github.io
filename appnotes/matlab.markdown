---
layout: page
title: MATLAB
parent: Application Notes
nav_order: 5
---

# MATLAB on Sulis
{: .no_toc }

1. TOC
{:toc}

[MATLAB](https://mathworks.com/products/matlab.html) is a programming and numeric computing platform used by millions of engineers and scientists to analyze data, develop algorithms, and create models.

## Accessing MATLAB

```bash
{{site.data.terminal.prompt}} module spider MATLAB
```

any MATLAB module can be loaded directly 

```bash
{{site.data.terminal.prompt}} module load MATLAB/2022a
```

## Running MATLAB
Please do NOT run MATLAB on the login nodes under any circumstances! MATLAB is a very resource-intensive application so it's important that you always follow the [Sulis Acceptable User Policy](../polices) and run MATLAB via the job scheduler. As discussed in [Job submission](../gettingstarted/batchq) section, one option is to launch an interactive session 

```bash
{{site.data.terminal.prompt}} salloc --account=suxxx-somebudget -N 1 -n 1 --cpus-per-task=4 --mem-per-cpu=3850 --time=8:00:00
```

after a successful resource allocation, the interactive session will bring to a computational node other than the login one. MATLAB itself can be run interactively  

```bash
[user@node042(sulis) ~]$ module purge && module load MATLAB/2022a
[user@node042(sulis) ~]$ matlab -nodesktop
```

with the result of launching the command-line interface. 
MATLAB can be used as script processing tool as well. For example, running a matlab script `example.m` can be done via

```bash
[user@node042(sulis) ~]$ matlab -batch "example"
```

Other than allocating an interactive session on Sulis nodes, it is also possible to launch a batch job asking MATLAB to evaluate a matlab script, e.g.
 
<p class="codeblock-label">submit.sbatch</p>
```bash
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=4
#SBATCH --mem-per-cpu=3850
#SBATCH --time=15:00:00
#SBATCH --account=suXXX-somebudget

module purge
module load MATLAB/2022a

# run the 'example.m' matlab file
srun matlab -batch "example"
```

## Parallel Computing Toolbox

MATLAB Parallel Computing Toolbox enables multiple MATLAB workers to be run on a single machine/compute node, therefore, providing you greater control over the parallelism of your application than is provided by the built-in multi-threading as described above. Two of the core parallel functions are `parfor` (or parallel FOR loops) and `spmd` (or single program multiple data).

<p class="codeblock-label">submit.sbatch</p>
```bash
#!/bin/bash
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=28
#SBATCH --mem-per-cpu=3850
#SBATCH --time=15:00:00
#SBATCH --account=suXXX-somebudget

module purge
module load MATLAB/2018b

srun matlab -batch "parallel_matlabscript"
```

To make sure you start the correct number of parallel workers (i.e. 1 worker per allocated compute core using the 'local' profile), you must make sure that the parpool command in your MATLAB script is written in the following way:

```matlab
parpool('local', str2num(getenv('SLURM_CPUS_PER_TASK')))
```

And you should release the MATLAB workers at the end using the

```matlab
delete(gcp);
```

command just before `exit;`.

## Parallel Computing Toolbox on Multiple Nodes
Using a low-latency network (Infiniband) is essential when distributing communicating MATLAB workers across multiple nodes. At the present moment (Q2 2023), MATLAB does not yet support Infiniband and a third-party library for Message-Passing-Interface (MPI) communications has to be invoked. This is achieved on Sulis by loading additional modules pointing to the MVAPICH2 (MPI) library and setting the required environment. The corresponding submission script then would transform into the following

<p class="codeblock-label">submit.sbatch</p>
```bash
#!/bin/bash
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=1
#SBATCH --time=15:00:00
#SBATCH --mem-per-cpu=3850
#SBATCH --account=suXXX-somebudget

module purge
module load MATLAB/2022a
module load iccifort/2019.5.281
module load MATLAB-parenv/0.0-iccifort-2019.5.281

# set the number of nodes to run MATLAB on
export MATLAB_NNODES=2
# number of tasks per node, giving NNODES*NTASKS_PER_NODE workers
export MATLAB_NTASKS_PER_NODE=16
# each worker can use multiple threads, check if it gives a better performance
export MATLAB_CPUS_PER_TASK=1
# export --account filed into a variable to read in MATLAB
export SLURM_ACCOUNT=suXXX-somebudget

# command available in the later version of MATLAB for launching the multinode_matlabscript.m file
matlab -batch "multinode_matlabscript"
```

Note, that we set `--cpus-per-task` to 1 here (increase its value when more memory is required), but introduced the `MATLAB_*` variables which would further initiate the parallel environment from the file below

<p class="codeblock-label">multinode_matlabscript.m</p>
```matlab
% add parallel environment/scripts files path
parenv_root = getenv('EBROOTMATLABMINPARENV');
addpath(strcat(parenv_root, '/scripts'));

% capture the parallelisation parameters from ones set in the batch submission script
thds = str2num(getenv('MATLAB_CPUS_PER_TASK'));
tsks = str2num(getenv('MATLAB_NTASKS_PER_NODE'));
nods = str2num(getenv('MATLAB_NNODES'));
acct = getenv('SLURM_ACCOUNT');

% cast into a string variable
submit_arguments = sprintf('-t 48:00:00 --mem-per-cpu=3850 --nodes=%d --ntasks-per-node=%d --cpus-per-task=%d --account=%s', nods, tsks, thds, acct);

% set the cluster profile
c = parallel.cluster.Slurm;
c.NumWorkers = tsks*nods;
c.NumThreads = thds;
c.SubmitArguments = submit_arguments;
c.CommunicatingJobWrapper = strcat(parenv_root, '/scripts/communicatingJobWrapper.sh')

% set and show the MPI library in use (has to be in MVAPICH2 location)
mpiLibConf

% Define the job using batch, which evaluates myFunc in parallel
job = c.batch('myFunc', 'Pool', c.NumWorkers-1);

% variables defined in `myFunc` can be accessed via
data = fetchOutputs(job);
```

Instead of `c.batch()` function, we could initiate the pool of workers with

```matlab
parpool(c, c.NumWorkers);
```
and place the parallel code (e.g., from the body of `myFunc.m` of the previous example) below this call.

## Limitations
We currently have noticed difficulties in launching on a large number of CPUs per node for parallel MATLAB programming (e.g., parallel pools). We advise setting this number below 32. If more workers are needed, one can run the calculation on multiple nodes as per examples above requesting the number of per-node workers `MATLAB_NTASKS_PER_NODE` less or equal to 32. 

