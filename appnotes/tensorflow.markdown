---
layout: page
title: TensorFlow
parent: Application Notes
nav_order: 2
---

# TensorFlow on Sulis
{: .no_toc }

1. TOC
{:toc}

These notes constitute a brief guide to using TensorFlow on Sulis, with emphasis on using the GPU hardware. They may update occasionally as newer software is deployed. 


## Accessing TensorFlow

We strongly recommend using the version of TensorFlow provided by the module system. This has been compiled specifically for the hardware in Sulis and subsequently subjected to verification tests.

To search for an appropriate version of TensorFlow

```bash
{{site.data.terminal.prompt}} module spider TensorFlow
```

This will list a number of TensorFlow builds that can be added into your environment. For example `TensorFlow/2.5.0`. Querying this version specifically will provide information on prerequisite modules than must first be loaded.

```bash
{{site.data.terminal.prompt}} module spider TensorFlow/2.5.0

-----------------------------------------------------------------------------
  TensorFlow: TensorFlow/2.5.0
-----------------------------------------------------------------------------
    Description:
      An open-source software library for Machine Intelligence


    You will need to load all module(s) on any one of the lines below before
    the "TensorFlow/2.5.0" module is available to load.

      GCC/10.2.0  CUDA/11.1.1  OpenMPI/4.0.5
      GCC/10.2.0  OpenMPI/4.0.5

```
In this case there are two sets of possible prerequisite modules. The first includes CUDA and should
be used for running TensorFlow on the Sulis GPU nodes. The second is for use on the standard
compute nodes.

TensorFlow can hence be added to your environment for GPU computation by loading the following modules 

```bash
{{site.data.terminal.prompt}} module load GCC/10.2.0 CUDA/11.1.1  OpenMPI/4.0.5
{{site.data.terminal.prompt}} module load TensorFlow/2.5.0
```

Loading a TensorFlow module in this way also adds additional prerequisites to your terminal
and Python environment. For example in this case the Python 3.8.6 module is loaded, along with a SciPy-bundle module that provides NumPy 1.19.4, SciPy 1.5.4 and Pandas 1.14. There is no need
to install these via pip. All dependencies of TensorFlow itself are provided.

NOTE : Attempting to use TensorFlow loaded in this way will fail unless running on an GPU-enabled
node in an interactive session or SLURM job script. 

## Compatibility

Many users will wish to use higher level software that either depends explicitly on TensorFlow or would ideally run in the same environment. Some such software is already available via the module system and can be queried using `module av` after loading TensorFlow as per above. For the example of TensorFlow/2.5.0 above:

```bash
{{site.data.terminal.prompt}} module av
```
The output of which includes (in this case)
```bash
OpenCV/4.5.1-contrib
bokeh/2.2.3
dask/2021.2.0
scikit-learn/0.23.2
```
and other possible modules which can loaded into the same environment as TensorFlow. Additional software *can* be [installed via pip](../gettingstarted/software/python/#accessing-additional-packages) into this environment. However we recommend discussing with your [research computing](../support) team first, particularly for software that will perform significant computations. Similarly requests to install additional packages as modules should be directed to your local support contact.

## Job submission

The following is an illustrative example of a job script which executes a TensorFlow 2 ResNet50
image classification benchmark on a single GPU when submitted with `sbatch` from within a clone of the repository at:

[https://github.com/aime-team/tf2-benchmarks](https://github.com/aime-team/tf2-benchmarks)

<p class="codeblock-label">tensorflow2.slurm</p>
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
module load GCC/10.2.0 CUDA/11.1.1
module load TensorFlow/2.5.0

srun python tf2-benchmarks.py --model resnet50 --enable_xla --batch_size 256 --num_gpus 1
```

Performance with this benchmark should be close to 980 images per second. 

## Performance and containerisation

TensorFlow performance critically depends on the underlying CUDA Deep Neural Network library [(cuDNN)](https://developer.nvidia.com/cudnn). Full support for the Nvidia A100 GPUs in Sulis is only present in cuDNN version 8 and later. Versions of TensorFlow provisioned via the module system
have been built to take advantage of this.

Users running containerised workflows via [Singularity](../advanced/containers.markdown) should ensure that 
container images use cuDNN 8 or later where possible. 

<!--- 

## TensorFlow 1.15

For workloads that have not yet migrated to TensorFlow 2 we provide modules for TensorFlow 1.15. These are based on builds from the [NVIDIA/tensorflow](https://github.com/NVIDIA/tensorflow) project
which supports newer hardware and improved libraries for NVIDIA GPU users using TensorFlow 1.x.  In particular they use cuDNN 8 for optimal performance on the A100 GPUs.

Installing TensorFlow 1.15 from PyPi using pip is not advisable. The PyPi builds are based on cuDNN 7 and give significantly worse performance. 

For example, in the standard ResNet50 classification benchmark:

| Build                            | fp 32 (images/second) | fp 16 (images/second) |
|----------------------------------|-----------------------|-----------------------|
| PyPi build (cuDNN 7)             |        625            |                       |
| TensorFlow/1.15 module (cuDNN 8) |        1027           |        2486           |



As with TensorFlow 2, we recommend containerised workflows involving TensorFlow 1 use cuDNN 8 or later.

--->








