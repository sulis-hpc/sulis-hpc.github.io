---
layout: page
title: Containerisation
parent: Advanced Topics
nav_order: 5
---

# Containers
{: .no_toc }

1. TOC
{:toc}

This topic explains how to use the Singularity container system on Sulis.

## Introduction

### What is Singularity

[Singularity](https://singularity.hpcng.org/) is a program that performs a type of [operating-system level virtualisation](https://en.wikipedia.org/wiki/OS-level_virtualization) known as containerisation. Singularity enables users to run workflows on the HPC clusters that would either need a different operating system to that being provided, or would otherwise be extremely difficult to build using our preferred build system ([EasyBuild](https://easybuilders.github.io/easybuild/)).

Singularity is capable of running many [Docker](https://www.docker.com) containers by automatically converting them to the stand-alone singularity file format.

### Additional resources

* [Singularity Documentation](https://singularity.hpcng.org/user-docs/master/index.html) - make sure you are viewing documentation that matches the version being used.
* [Singularity Container Library](https://cloud.sylabs.io/library)
* [Docker Hub](https://hub.docker.com/)
* [NVidia NGC Catalog](https://catalog.ngc.nvidia.com/) - catalog of containers maintained by NVidia.

## Using Singularity
### Important notes

* It is not possible to build containers on Sulis. However it is possible to use containers built elsewhere.
* When you "pull" existing Singularity containers, the singularity image for the container will be downloaded.
* When you "pull" Docker images (for example from [DockerHub](https://hub.docker.com/)), the "layers" of the Docker image will be downloaded, combined into a single image and automatically converted into the Singularity image format (.sif) which will make some additional changes for compatibility reasons. We cannot, of course, guarantee compatibility with all Docker container images.

### Accessing the Singularity commands

Singularity is available on the compute, gpu and hmem nodes and you can run it within [interactive sessions]({% link gettingstarted/batchq/interactive.markdown %}) by simply running the `singularity` command; there are no modules to load and the version available will be updated on a semi-regular basis to keep in-line with security updates.

Running the `singularity` command with no options gives a list of available commands:

```terminal
[user@node046(sulis) ~]$ singularity
Usage:
  singularity [global options...] <command>

Available Commands:
  build       Build a Singularity image
  cache       Manage the local cache
  capability  Manage Linux capabilities for users and groups
  completion  generate the autocompletion script for the specified shell
  config      Manage various singularity configuration (root user only)
  delete      Deletes requested image from the library
  exec        Run a command within a container
  inspect     Show metadata for an image
  instance    Manage containers running as services
  key         Manage OpenPGP keys
etc...
```

So running

```terminal
[user@node046(sulis) ~]$ singularity version
3.8.4-1.el8
```

shows the version number.

### Finding and downloading images

The singularity container image is the physical representation of the container environment (such as the operating system, installed packages and customised scripts). Container images can be downloaded from the [Singularity Library](https://cloud.sylabs.io/library), [Docker Hub](https://hub.docker.com/) or [NVidia NGC Catalog](https://catalog.ngc.nvidia.com/).

For the convenience of users, several containers have been pre-downloaded (and converted in the case of Docker images) and may be found at `/sulis/containers/`; users should be aware that these containers may not be the latest versions available from the corresponding Singularity Library or Docker Hub and therefore may not have the latest updates.

The `singularity search` command can be used to search for images in the Singularity Library e.g. (again, from within an [interactive session]({% link gettingstarted/batchq/interactive.markdown %}))

```terminal
[user@node046(sulis) ~]$ singularity search ubuntu
Found 144 container images for amd64 matching "ubuntu":

        library://amit112amit/default/ubuntu_numba:v0.0

        library://artinmajdi/default/cuda_ubuntu1604.sif:version0.2
                Signed by: E35C69E9D71B69856036D151103B9AF8ECB6192C,E35C69E9D71B69856036D151103B9AF8ECB6192C

        library://artinmajdi/default/cuda_ubuntu1604:sha256.612ef71a16585c2885411c8dc299725e7b4d993426a9e47636c84a3ddba74776
                Signed by: E35C69E9D71B69856036D151103B9AF8ECB6192C

        library://brendanharding/ubuntu_openmpi/ubuntu-18.04_openmpi-4.0.5:latest

        library://brx/test/ubuntu1:latest

        library://cfbasz/default/ubuntu_remix:latest,v0.1.2
                Performed a full system update and added some missing packages (gdb, gdb-doc and cgdb).
                Signed by: 97710305c7fce302aea7efebd029be270a153de0
...etc.
```

A particular image can then be downloaded or "pulled" using the `pull` command e.g. and will be (by default) saved to your current directory with a filename ending in `.sif` e.g.:

```terminal 
[user@node046(sulis) ~]$ singularity pull library://library/default/ubuntu:21.04
INFO:    Downloading library image
35.5MiB / 35.5MiB [==============================================================================================================================================================] 100 % 915.2 KiB/s 0s
[user@node046(sulis) ~]$
```

This downloads the operating system image for Ubuntu 21.04 and saves it in the Singularity Image Format (.sif which is a compressed, read-only format) in your current directory; this singularity images has been pre-downloaded and may be found at `/sulis/containers/cloud.sylabs.io/library/default/ubuntu/ubuntu_21.04.sif`. We would generally recommend downloading "official" singularity images such as those starting `library://library/default/`.

Docker images can be found by searching the [Docker Hub](https://hub.docker.com/) online and then using the `pull` command again which will download the layers of the Docker container and automatically convert them into a single Singularity Image Format e.g.

```terminal
[user@node046(sulis) ~]$ singularity pull docker://ubuntu:18.04
INFO:    Converting OCI blobs to SIF format
INFO:    Starting build...
Getting image source signatures
Copying blob 284055322776 done  
Copying config 04dc6f46a4 done  
Writing manifest to image destination
Storing signatures
2021/11/19 09:39:15  info unpack layer: sha256:284055322776031bac33723839acb0db2d063a525ba4fa1fd268a831c7553b26
INFO:    Creating SIF file...
[user@node046(sulis) ~]$
```

This Docker image has been pre-downloaded and converted to singularity, it may be found at `/sulis/containers/hub.docker.com/ubuntu/ubuntu_18.04.sif`.

It is important to note that in the process of converting the Docker image to the Singularity image, some changes are made to the image to deal with compatibility differences between the two technologies. It is therefore possible that some Docker Images may not run correctly (or at all) when converted to a Singularity image. In these circumstances it may be necessary to build a Singularity image from scratch by following the [documentation](https://singularity.hpcng.org/user-docs/master/index.html).

### Running Singularity containers

The following examples use the lolcow Singularity container image (provided by [Sylabs](https://sylabs.io/) for example purposes) which consists of a [Ubuntu](https://ubuntu.com/) base operating system with the [cowsay](https://en.wikipedia.org/wiki/Cowsay) program installed within it.

Start an [interactive session](({% link gettingstarted/batchq/interactive.markdown %})) on a compute node and then run the following command to download the image to your local directory:

```terminal
[user@node046(sulis) ~]$ singularity pull library://sylabsed/examples/lolcow
INFO:    Downloading library image
79.9MiB / 79.9MiB [================================================================================================================================================================] 100 % 2.5 MiB/s 0s
WARNING: integrity: signature not found for object group 1
WARNING: Skipping container verification
[user@node046(sulis) ~]$
```

This should leave you with the image file called `lolcow_latest.sif` in your current working directory. You may also copy a pre-downloaded version to your home directory from `/sulis/containers/cloud.sylabs.io/sylabsed/examples/lolcow_latest.sif`.

#### Starting a shell

Since a singularity image contains an Operating System environment, it is possible to start a shell session in an image (within an [interactive session]({% link gettingstarted/batchq/interactive.markdown %}) on a compute node):

```terminal
[user@node046(sulis) ~]$ singularity shell lolcow_latest.sif
Singularity> cat /etc/os-release 
NAME="Ubuntu"
VERSION="18.10 (Cosmic Cuttlefish)"
ID=ubuntu
ID_LIKE=debian
PRETTY_NAME="Ubuntu Cosmic Cuttlefish (development branch)"
VERSION_ID="18.10"
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
VERSION_CODENAME=cosmic
UBUNTU_CODENAME=cosmic
Singularity> exit
exit
[user@node046(sulis) ~]$
```

Once "inside" the container, you are the same user that you are on the host system.

#### Executing a command within a container

It is also possible to execute a command within a container, without starting a shell.

```terminal
[user@node046(sulis) ~]$ singularity exec lolcow_latest.sif cowsay moo
 _____
< moo >
 -----
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
[user@node046(sulis) ~]$
```

#### Using Singularity within a batch job

The standard instructions for [single node jobs]({% link gettingstarted/batchq/singlenode.markdown %}) can be followed for writing a batch script in order to execute a singularity based job on the cluster. The following example batch script demonstrates how it would be possible to run `cowsay` via the singularity container within a compute job (allocating 8 cores to the job).

<p class="codeblock-label">singularity.slurm</p>
```bash
#!/bin/bash
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=8
#SBATCH --mem-per-cpu=3850
#SBATCH --time=08:00:00
#SBATCH --account=suxxx-somebudget

srun singularity exec lolcow_latest.sif cowsay -f tux Hello
```

Once your job has run, you should have a slurm output file containing the following:

```
 _______
< Hello >
 -------
   \
    \
        .--.
       |o_o |
       |:_/ |
      //   \ \
     (|     | )
    /'\_   _/`\
    \___)=(___/
```

#### Using Singularity with GPUs

Singularity supports enabling a running container to see and work with GPUs. In order to do this, you must pass the `--nv` option to singularity while [running on a GPU node]({% link gettingstarted/batchq/gpu.markdown %}), for example:

<p class="codeblock-label">singularity-gpu.slurm</p>
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

srun singularity run --nv ./tensorflow_21.10-tf1-py3.sif python3 ./tf1-benchmarks/scripts/tf_cnn_benchmarks/tf_cnn_benchmarks.py --model resnet50 --num_gpus=1 --xla --batch_size=256
```

This would run [TensorFlow 1 benchmarks](https://github.com/aime-team/tf1-benchmarks) (see also the [TensorFlow Application Notes]({% link appnotes/tensorflow.markdown %})) on the [NVidia Tensorflow 21.10-tf1-py3](https://catalog.ngc.nvidia.com/orgs/nvidia/containers/tensorflow/tags) Docker container that has been converted to the singularity format (this container has been pre-downloaded and may be found at `/sulis/containers/catalog.ngc.nvidia.com/nvidia/tensorflow\tensorflow_21.10-tf1-py3.sif`).

### Working with files

When you start a singularity container, the following locations are automatically made available (from the host system) within it:

* Your current working directory. You still start in this directory when the containers runs, and it is additionally accessible via the `$PWD` environment variable.
* Your home directory. You can access files in this directory within the container in the same way that you access files on the host system and it is additionally accessible via the `$HOME` environment variable. Note also that for most users, the current working directory will be within your home directory.
* `/tmp` from the host computer. This will be made available in the same location within the container.

## Frequently Asked Questions

### How do I send multiple commands to singularity exec?

The recommended way of sending multiple commands to a container via `singularity exec` is to write them in a script file. For example (using the earlier cowsay example), in order to pipe the output from the [fortune](https://en.wikipedia.org/wiki/Fortune_(Unix)) command to the `cowsay` command, you would write the following script:

<p class="codeblock-label">fortune-cowsay.sh</p>
```bash
#!/bin/bash
fortune | cowsay -f elephant
```

Assuming the script is called `fortune-cowsay.sh`, make it executable using the `chmod` command:

```terminal
{{site.data.terminal.prompt}} chmod u+x fortune-cowsay.sh
```

and then write a submission script to use the new script:

<p class="codeblock-label">fortune-cowsay.slurm</p>
```bash
#!/bin/bash
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=8
#SBATCH --mem-per-cpu=3850
#SBATCH --time=08:00:00
#SBATCH --account=suxxx-somebudget

srun singularity exec lolcow_latest.sif ./fortune-cowsay.sh
```

Once your job has run, you should have a slurm output file containing something like:

```
 ______________________________________
/ Q: What's a light-year? A: One-third \
\ less calories than a regular year.   /
 --------------------------------------
 \     /\  ___  /\
  \   // \/   \/ \\
     ((    O O    ))
      \\ /     \ //
       \/  | |  \/ 
        |  | |  |  
        |  | |  |  
        |   o   |  
        | |   | |  
        |m|   |m|
```

### What is my user name within the container?

Once you are "inside" the container, you are the same user as you are on the host system.

### Is it better to build my software or use a provided container?

We generally recommend that you build your software (or [request software to be built for you]({% link support.markdown %})) from source code rather than using a provided container. Software that is built for the HPC cluster it is run on will often perform better than software built elsewhere. However some software/toolchains can be very difficult to build from scratch and in these circumstances it may be better to use a provided container.

## Troubleshooting

### Singularity instance won't clear from list

If you started a singularity container as an instance (using `singularity instance start`) and the instance was not shut down cleanly, then the information relating to that instance will not be automatically removed. If you subsequently run the `singularity instance list` command on the same host you will see the instance despite it not running. This "state" information is stored in `.singularity/instances/sing/hostname` under your home directory. You should be able to delete the directory corresponding to the host and instance that you started.
