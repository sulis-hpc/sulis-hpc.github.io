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

Sulis contains a number of nodes equipped with Nvidia A100 GPUs. These are the 40GB variant of the A100, connected via PCI-express. Three A100s are installed in each node. GPU nodes are accessed via submitting batch scripts to the {{site.data.slurm.gpunode_partition_name}} partition. Such scripts should request one or more GPUs in their SLURM resource request.

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
{{site.data.terminal.prompt}} nvcc --gpu-code=sm_80 cuda_hello.cu
```
</details>

The following example job script requests a single GPU via the {{site.data.slurm.gpunode_partition_name}} partition. Each GPU nodes contains {{site.data.slurm.gpunode_cores_per_node}} which does not divide equally over the {{site.data.slurm.gpunode_gpus_per_node}} A100 GPUs in each node. We therefore recommend that single GPU jobs request 42 CPUs per task. 

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

module purge
module load {{site.data.software.defaultgcc}} {{site.data.software.defaultcuda}}

srun ./a.out
```

The resource request `gres=gpu:{{site.data.slurm.gpunode_gpu_gres_name}}:1` specifies that we require a single A100 GPU for the job. The request `partition={{site.data.slurm.gpunode_partition_name}}` overrides the default partition and tells SLURM the job should run on the partition consisting of GPU enabled nodes. BOTH are required.

In this example, a more complicated program than our `cuda_hello.cu` might be able to make use of the 42 CPUs/cores by spawning additional threads. Some codes (e.g. LAMMPS) instead benefit from multiple MPI tasks sharing a single CPU, in which case the `ntasks-per-node` part of the resource request should reflect the desired number of tasks and `cpus-per-task` reduced such that the total number of CPUs/cores requested is no more than 42.

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
Device 0 : Ampere A100
Using device 0
```
Note that our program only identified a single G

</details>

## Single GPU jobs in Python

### GPU accelerated Python packages

Some Python codes may make use GPU acceleration. Most commonly this will be via use of GPU-accelerated packages such as TensorFlow, PyTorch, Magma and many others. Jobs scripts should ensure that the appropriate version of [environment modules](../software/modules) are loaded, i.e. those which have a CUDA module as a dependency. 

A suitable script is below, using TensorFlow as an example.

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

module purge
module load {{site.data.software.defaultgcc}} {{site.data.software.defaultcuda}} {{site.data.software.defaultmpi}}
module load TensorFlow/2.4.1 

srun python tf_gpu.py
```
Note that omitting {{site.data.software.defaultcuda}} from the first `module load` command would use a toolchain that is not GPU enabled. The second `module load` would then import a build of TensorFlow which is not GPU enabled. 

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

module purge
module load {{site.data.software.defaultgcc}} {{site.data.software.defaultcuda}} {{site.data.software.defaultmpi}}
module load CuPy/8.5.0 

srun python cupy_api.py
```

Here the 42 CPUs/cores available to the job (or some subset thereof) could be used for a multiprocessing pool or a set of workers which all share access to the single GPU. This might be appropriate for workloads in which a set of serial calculations benefit from GPU acceleration but cannot effectively make use of a whole A100.

## Single node, multi-GPU 

Some scientific packages will support use of multiple GPUs out of the box and handle assignment of GPUs to tasks or CPUs/threads automatically or via use input to the software.

In other cases you may need to provide additional information to `srun` to indicate how the available GPUs should be shared across the elements of your calculation.

For example, you may wish to use a whole GPU node to create a Python multiprocessing pool of 18 single-CPU workers which equally share the available 3 GPUs within a node.

srun -n 1 -G 3 -c 18 --cpus-per-gpu=6 python my_thing.py

Note that the cpus-per-task part of the resource request is still set to 128 such that the 18 workers have all the memory in the node available.

Alternatively you may have an MPI program in which each of 3 single-CPU tasks can effectively utilise an entire GPU.

srun -n 3 -G 3 -c 1 --gpus-per-task=1 my_mpi_thing

More advanced options give precise control over which tasks/CPUs are allocated to which GPU. See the srun section of the SLURM manual.

## Multi-node GPU jobs
