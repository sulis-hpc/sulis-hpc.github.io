---
layout: page
title: GPU jobs 
parent: Job submission
grand_parent: Getting Started
nav_order: 5
---

# GPU jobs
{: .no_toc }

1. TOC
{:toc}

Sulis contains 30 nodes equipped with Nvidia A100 GPUs. These are the 40GB variant of the A100, connected via PCI-express. Three A100s are installed in each node. In addition there are 18 nodes equipped with three Nvidia L40 GPUs (48GB). 

{: .note }
These are PCI-e GPU cards and are not connected via Nvidia SXM. They are unlikely to be suitable for training large models that need to span the memory of multiple GPUs simultaneously. 

{: .info}
The L40 GPU nodes will be available sometime in September 2024 following configuration and testing.

GPU nodes are accessed via submitting batch scripts to the {{site.data.slurm.gpunode_partition_name}} partition. Such scripts should request one or more GPUs in their SLURM resource request.

Most GPU jobs will require loading of the a CUDA [environment module](../software/modules) to make use of GPU acceleration. 

## Single GPU CUDA jobs

See the [compiling section](../software/compiling) for information on compiling CUDA C codes on Sulis.

<details markdown="block" class="detail">
  <summary>An example GPU program in CUDA C.</summary>
An trivial example of a CUDA C program.

<p class="codeblock-label">cuda_hello.cu</p>
```c 
#include <stdio.h>
#include <cuda.h> 

int main() {

  printf("Hello world from the host\n");
  printf("Checking for CUDA devices...\n");


  int count;
  cudaError_t err;
  err = cudaGetDeviceCount(&count);
  if ( (count==0) || (err!=cudaSuccess) ) {
    printf("No CUDA supported devices are available in this system.\n");
    exit(EXIT_FAILURE);
  } else {
    printf("Found %d CUDA device(s) in this system\n",count);
  }

  cudaDeviceProp prop;
  int idev;
  for (idev=0;idev<count;idev++) {

    // Call another CUDA helper function to populate prop
    err = cudaGetDeviceProperties(&prop,idev);
    if ( err!=cudaSuccess ) {
      printf("Error getting device properties\n");
      exit(EXIT_FAILURE);
    }

    printf("Device %d : %s\n",idev,prop.name);

  }

  err = cudaGetDevice(&idev);
  if ( err!=cudaSuccess ) {
    printf("Error identifying active device\n");
    exit(EXIT_FAILURE);
  }
  printf("Using device %d\n",idev);

  return(EXIT_SUCCESS);

}
``` 
This might be compiled into the executable `a.out` via:
```bash
{{site.data.terminal.prompt}} module load {{site.data.software.defaultgcc}} {{site.data.software.defaultcuda}}
{{site.data.terminal.prompt}} nvcc -arch=sm_80 -arch=sm_89 cuda_hello.cu
```
Note that we've specified the compute capability of both GPU types when compiling. 

</details>

The following example job script requests a single GPU via the {{site.data.slurm.gpunode_partition_name}} partition. Each GPU node contains {{site.data.slurm.gpunode_cores_per_node}} cores which does not divide equally over the {{site.data.slurm.gpunode_gpus_per_node}} A100 GPUs in each node. We therefore recommend that single GPU jobs request 42 CPUs per task. 

<p class="codeblock-label">gpu.slurm</p>
```bash
#!/bin/bash
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=42
#SBATCH --mem-per-cpu={{site.data.slurm.gpunode_ram_per_core}}
#SBATCH --gres=gpu:{{site.data.slurm.gpunode_gpu_gres_name}}:1
#SBATCH --partition={{site.data.slurm.gpunode_partition_name}}
#SBATCH --time=08:00:00
#SBATCH --account=suxxx-somebudget

module purge
module load {{site.data.software.defaultgcc}} {{site.data.software.defaultcuda}}

srun ./a.out
```

The resource request `gres=gpu:{{site.data.slurm.gpunode_gpu_gres_name}}:1` specifies that we require a single A100 GPU for the job. This could be substituted with `gres=gpu:{{site.data.slurm.gpunode_gpu_gres_name2}}:1` to request a single L40 GPU instead. Alternative, use `gres=gpu:1` to indicate that a GPU of any type if sufficient.

The request `partition={{site.data.slurm.gpunode_partition_name}}` overrides the default partition and tells SLURM the job should run on the partition consisting of GPU enabled nodes. 

In this example, a more complicated program than our `cuda_hello.cu` might be able to make use of the 42 CPUs/cores by spawning additional threads. Some codes (e.g. LAMMPS) instead benefit from multiple MPI tasks sharing a single GPU, in which case the `ntasks-per-node` part of the resource request should reflect the desired number of tasks and `cpus-per-task` reduced such that the total number of CPUs/cores requested is no more than 42.

GPU jobs are submitted to SLURM in the usual way.

```shell
{{site.data.terminal.prompt}} sbatch gpu.slurm
Submitted batch job 212981
```

<details markdown="block" class="detail">
  <summary>Output from example program.</summary>
```shell
{{site.data.terminal.prompt}} cat slurm-212981.out
Hello world from the host
Checking for CUDA devices...
Found 1 CUDA device(s) in this system
Device 0 : A100-PCIE-40GB
Using device 0
```
Note that our program only identified a single GPU despite their being 3 in the node. Only the GPU allocated to us by SLURM is visible.

</details>

## Single GPU jobs in Python

### GPU accelerated Python packages

Some Python codes may make use GPU acceleration. Most commonly this will be via use of GPU-accelerated packages such as TensorFlow, PyTorch, Magma and many others. Jobs scripts should ensure that the appropriate version of [environment modules](../software/modules) are loaded, i.e. those which have a CUDA module as a dependency or have CUDA suffixed to the module name. 

A suitable script is below, using PyTorch as an example.

<details markdown="block" class="detail">
  <summary>Python script <code>torch_gpu.py</code>  to test for GPU support in PyTorch.</summary>
This trivial script imports the PyTorch package and checks if the imported build
of has GPU acceleration available.

<p class="codeblock-label">torch_gpu.py</p>
```python
import torch

if torch.torch.cuda.is_available():
    print("Imported PyTorch package was built with GPU support")
else:
    print("Imported PyTorch package was NOT built with GPU support")
``` 
</details>

This can be executed on a GPU node with the following SLURM job script.

<p class="codeblock-label">torch_gpu.slurm</p>
```bash
#!/bin/bash
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=42
#SBATCH --mem-per-cpu={{site.data.slurm.gpunode_ram_per_core}}
#SBATCH --gres=gpu:1
#SBATCH --partition={{site.data.slurm.gpunode_partition_name}}
#SBATCH --time=08:00:00
#SBATCH --account=suxxx-somebudget

module purge
module load {{site.data.software.defaultfoss}}
module load PIP-PyTorch/{{site.data.software.PyTorchversion}}

srun python torch_gpu.py
```

This script should report a positive result!

<!-->

<details markdown="block" class="detail">
  <summary>Python script <code>tf_gpu.py</code>  to test for GPU support in TensorFlow.</summary>
This trivial script imports the TensorFlow package and checks if the imported build
of TensorFlow is built to use GPU acceleration.

<p class="codeblock-label">tf_gpu.py</p>
```python
import tensorflow as tf

if tf.test.is_built_with_cuda():
    print("Imported TensorFlow package was built with GPU support")
else:
    print("Imported TensorFlow package was NOT built with GPU support")
``` 
</details>

This can be executed on a GPU node with the following SLURM job script.

<p class="codeblock-label">tf_gpu.slurm</p>
```bash
#!/bin/bash
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=42
#SBATCH --mem-per-cpu={{site.data.slurm.gpunode_ram_per_core}}
#SBATCH --gres=gpu:{{site.data.slurm.gpunode_gpu_gres_name}}:1
#SBATCH --partition={{site.data.slurm.gpunode_partition_name}}
#SBATCH --time=08:00:00
#SBATCH --account=suxxx-somebudget

module purge
module load {{site.data.software.defaultgcc}} {{site.data.software.defaultcuda}} {{site.data.software.defaultmpi}}
module load TensorFlow/2.5.0 

srun python tf_gpu.py
```
Note that omitting {{site.data.software.defaultcuda}} from the first `module load` command would use a toolchain that is not GPU enabled. The second `module load` would then import a build of TensorFlow which is not GPU enabled. 

-->

### Using GPUs directly in Python code


Some workflows may involve GPU-accelerated code written in Python. This may take the form of Python functions executed as kernels on the GPU device using [Numba](https://numba.pydata.org/), or drop-in replacements for compute-intensive NumPy and SciPy operations such as those implemented in [CuPy](https://cupy.dev/). These can be executed in job scripts provided the appropriate packages are loaded as [environment modules](../software/modules). 

<details markdown="block" class="detail">
  <summary>Example script using CuPy interace to CUDA <code>cupy_api.py</code>.</summary>
This script replicates the compiled CUDA C example above.

<p class="codeblock-label">cupy_api.py</p>
```python
import cupy as cp

print("Hello world from the host")
print("Checking for CUDA devices...")

count = cp.cuda.runtime.getDeviceCount()

if count<1:
    print("No CUDA supported devices are available in this system.")
else:
    print("Found %d CUDA device(s) in this system." % count)
    
for idev in range(count):
    prop = cp.cuda.runtime.getDeviceProperties(idev)        
    print("Device %d %s" % (idev, prop['name'].decode()));

idev = cp.cuda.runtime.getDevice();
print("Using device %d" %idev);
``` 
</details>

The following SLURM job script is suitable for a Python code written to use a single GPU in CuPy. Other packages such as [Numba](https://numba.pydata.org/) or [PyCUDA](https://documen.tician.de/pycuda/) might be used instead. 

<p class="codeblock-label">cupy.slurm</p>
```bash
#!/bin/bash
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=42
#SBATCH --mem-per-cpu={{site.data.slurm.gpunode_ram_per_core}}
#SBATCH --gres=gpu:{{site.data.slurm.gpunode_gpu_gres_name}}:1
#SBATCH --partition={{site.data.slurm.gpunode_partition_name}}
#SBATCH --time=08:00:00
#SBATCH --account=suxxx-somebudget

module purge
module load {{site.data.software.defaultgcc}} {{site.data.software.defaultmpi}}
module load {{site.data.software.CuPy}}

srun python cupy_api.py
```

Here the 42 CPUs/cores available to the job (or some subset thereof) could be used for a multiprocessing pool or a set of workers which all share access to the single GPU. This might be appropriate for workloads in which a set of serial calculations benefit from GPU acceleration but cannot effectively make use of a whole A100.

## Single node, multi-GPU 

Some scientific packages will support use of multiple GPUs out of the box and handle assignment of GPUs to tasks or CPUs/threads automatically or via user input to the software.

In other cases you may need to provide additional information to `srun` to indicate how the available GPUs should be shared across the elements of your calculation. Two examples follow, which are not intended to be exhaustive. The examples use CuPy to interact with the GPU for illustrative purposes, but other methods will likely be more appropriate in many cases.

### Multiprocessing pool with shared GPUs

This example uses a whole GPU node to create a Python multiprocessing pool of 18 workers which equally share the available 3 GPUs within a node.

<details markdown="block" class="detail">
  <summary>Example <code>mp_gpu_pool.py</code>.</summary>
This trivial example demonstrates a multiprocessing pool in which the available GPUs are shared equally across the pool. Note that the programmer must calculate which device `idev` is to be used by which member of the pool `procid` and set that device as active for the current process. The function `f(i)` returns which processor in the pool and which GPU was used.

The number of processors available for the pool is set by interrogating the environment variable `SLURM_CPUS_PER_TASK`.

<p class="codeblock-label">mp_gpu_pool.py</p>
```python
import sys
import os
import multiprocessing as mp
import cupy as cp

def f(i):

    ngpus = cp.cuda.runtime.getDeviceCount()
    proc  = mp.current_process()
    procid = proc._identity[0]    
    idev   = procid%ngpus # which gpu device to use
    cp.cuda.runtime.setDevice(idev)
    print("proc %s processing input %d using GPU %d" % (proc.name, i, idev))
    return (procid, idev)
     
if __name__ == '__main__':

    p = int(os.environ['SLURM_CPUS_PER_TASK'])
    input_list = range(100)
    
    # Evaluate f for all inputs using a pool of processes
    with mp.Pool(p) as my_pool:
        print(my_pool.map(f, input_list))
```
</details>

In SLURM terminology this is a single task, using 18 CPUs and 3 GPUs. Note that the following script sets `cpus-per-task=42` in the resource request so that the pool of 18 processes has the entire RAM of the node available. 

<p class="codeblock-label">gpu_pool.slurm</p>
```bash
#!/bin/bash
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=42
#SBATCH --mem-per-cpu={{site.data.slurm.gpunode_ram_per_core}}
#SBATCH --gres=gpu:{{site.data.slurm.gpunode_gpu_gres_name}}:3
#SBATCH --partition={{site.data.slurm.gpunode_partition_name}}
#SBATCH --time=08:00:00
#SBATCH --account=suxxx-somebudget

module purge
module load {{site.data.software.defaultgcc}} {{site.data.software.defaultmpi}}
module load {{site.data.software.CuPy}} 

srun -n 1 -G 3 -c 18 --cpus-per-gpu=6 python mp_gpu_pool.py
```
As per earlier examples, one could replace `{{site.data.slurm.gpunode_gpu_gres_name}}` with `{{site.data.slurm.gpunode_gpu_gres_name2}}` to request L40 rather than A100 GPUs, or simply specify `--gres=gpu:3` to request three GPUs of either type.


### MPI application with one GPU per task

Alternatively you may have an MPI program in which each of 3 tasks can effectively utilise an entire GPU. 

<details markdown="block" class="detail">
  <summary>An example MPI GPU program in Python <code>mpi_gpu.py</code>.</summary>
Here each MPI task uses the GPU with id equal its rank.

<p class="codeblock-label">mpi_gpu.py</p>
```python
from mpi4py import MPI
import cupy as cp

comm = MPI.COMM_WORLD
my_rank = comm.Get_rank()
idev = my_rank

cp.cuda.runtime.setDevice(idev)
prop = cp.cuda.runtime.getDeviceProperties(idev)

print("MPI rank %d using GPU : %s_%d" % (my_rank, prop['name'].decode(),idev))

MPI.Finalize()
``` 
</details>


<p class="codeblock-label">mpi_gpu.slurm</p>
```bash
#!/bin/bash
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=3
#SBATCH --cpus-per-task=42
#SBATCH --mem-per-cpu={{site.data.slurm.gpunode_ram_per_core}}
#SBATCH --gres=gpu:{{site.data.slurm.gpunode_gpu_gres_name}}:3
#SBATCH --partition={{site.data.slurm.gpunode_partition_name}}
#SBATCH --time=08:00:00
#SBATCH --account=suxxx-somebudget

module purge
module load {{site.data.software.defaultgcc}} {{site.data.software.defaultmpi}}
module load {{site.data.software.mpi4pymodule}}
module load {{site.data.software.CuPy}}

srun -n 3 -G 3 --gpus-per-task=1 python mpi_gpu.py
```
Here each MPI task uses only one of the 42 CPUs allocated by task by SLURM, and one GPU. In other scenarios it might be sensible for each MPI task to use multiple CPUs via threading of spawning of subprocesses. See the [hybrid jobs](hybrid) section for more information.


## Multi-node GPU jobs

Jobs using more than three GPUs are possible by making a SLURM resource request for multiple nodes in the `{{site.data.slurm.gpunode_partition_name}}` partition. Users should be aware of the following.

- Use of multiple GPU nodes may be desirable to increase concurrency when processing a large batch of smaller calculations which collectively constitute a single job/workflow. This might be accomplished in a number of ways, e.g. a loosely coupled MPI application or a Python script which uses Dask to distribute independent calculations over a pool of GPU-enabled resource. See the [Ensemble Computing](../../advanced/ensemble) section of this documentation for examples.

- The GPU hardware configuration in Sulis is not optimised nor intended for workloads which require very high bandwidth communication between multiple  GPUs. Other tier 2 services, in particular [JADE II](https://www.jade.ac.uk/), [Baskerville](https://www.baskerville.ac.uk/) or [Bede](https://n8cir.org.uk/supporting-research/facilities/nice/) are much more likely to be appropriate for such workflows.

