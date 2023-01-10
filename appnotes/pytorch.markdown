---
layout: page
title: PyTorch
parent: Application Notes
nav_order: 7
---

# PyTorch on Sulis
{: .no_toc }

1. TOC
{:toc}

These notes constitute a brief guide to using [PyTorch](https://pytorch.org/) on Sulis, with emphasis on using the GPU hardware. They may update occasionally as newer software is deployed. 

## Accessing PyTorch

We strongly recommend using the version of PyTorch provided by the module system. This has been compiled specifically for the hardware in Sulis and subsequently subjected to verification tests.

To search for an appropriate version of PyTorch

```bash
{{site.data.terminal.prompt}} module spider PyTorch
```

This will list PyTorch builds that can be added into your environment. For example `PyTorch/1.9.0`. Querying this version specifically will provide information on prerequisite modules than must first be loaded.

```bash
{{site.data.terminal.prompt}} module spider PyTorch/1.9.0

-----------------------------------------------------------------------------
  PyTorch: PyTorch/1.9.0
-----------------------------------------------------------------------------
    Description:
      ATensors and Dynamic neural networks in Python with strong GPU 
      acceleration. PyTorch is a deep learning framework that puts Python 
      first.

    You will need to load all module(s) on any one of the lines below before
    the "PyTorch/1.9.0" module is available to load.

      GCC/10.2.0  CUDA/11.1.1  OpenMPI/4.0.5
      GCC/10.2.0  OpenMPI/4.0.5

```
In this case there are two sets of possible prerequisite modules. The first includes CUDA and should
be used for running PyTorch on the Sulis GPU nodes. The second is for use on the standard
compute nodes.

PyTorch can hence be added to your environment for GPU computation by loading the following modules 

```bash
{{site.data.terminal.prompt}} module purge
{{site.data.terminal.prompt}} module load GCC/10.2.0 CUDA/11.1.1  OpenMPI/4.0.5
{{site.data.terminal.prompt}} module load PyTorch/1.9.0
```

Loading a PyTorch/1.9.0 module in this way also adds additional prerequisites to your terminal
and Python environment. For example in this case the Python 3.8.6 module is loaded, along with a SciPy-bundle module that provides NumPy 1.19.4, SciPy 1.5.4 and Pandas 1.14. There is no need
to install these via pip. All dependencies of PyTorch itself are provided.

NOTE : Attempting to use PyTorch loaded in this way will fail unless running on an GPU-enabled
node in an interactive session or SLURM job script. 

## Torchvision, Torchtext etc

Many projects using PyTorch will make use of the associated Torchvision and Torchtext Python 
packages. These must be compiled in the same way as PyTorch, and are provisioned via the module
system for compatibility. Installing these packages via pip will create incompatibility. 

After loading the PyTorch modules as above, `module av` will list the modules available to
load into the same environment. 

```bash
{{site.data.terminal.prompt}} module av
```
The output of which includes (in this case)
```bash
torchtext/0.10.0-PyTorch-1.9.0
torchvision/0.10.0-PyTorch-1.9.0
```
and other possible modules which can loaded into the same environment. If additional components
of the PyTorch project are needed for your research then please request these via your [research computing](../support) team first.

## Job submission

The following is an illustrative example of a job script which executes a PyTorch ResNet50
image classification benchmark on a single GPU when submitted with `sbatch` from within a clone of the repository at:

[https://github.com/aime-team/pytorch-benchmarks](https://github.com/aime-team/pytorch-benchmarks)

<p class="codeblock-label">pytorch.slurm</p>
```bash
#!/bin/bash
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=42
#SBATCH --mem-per-cpu=3850
#SBATCH --gres=gpu:ampere_a100:1
#SBATCH --partition=gpu
#SBATCH --time=01:00:00
#SBATCH --account=suxxx-somebudget

module purge
module load GCC/10.2.0  CUDA/11.1.1  OpenMPI/4.0.5
module load PyTorch/1.9.0
module load torchvision/0.10.0-PyTorch-1.9.0

srun python main.py --model resnet50 --batch_size 256 --num_gpus 1
```

Performance with this benchmark should be close to 790 images per second. 

