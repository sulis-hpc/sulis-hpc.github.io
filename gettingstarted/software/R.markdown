---
layout: page
title: R 
parent: Software
grand_parent: Getting Started
nav_order: 5
---

# R 
{: .no_toc }

1. TOC
{:toc}

## R environment modules

Use `module spider R` to query the available R builds, and then load using (for example):

```shell
{{site.data.terminal.prompt}} module load {{site.data.software.Rtoolchain}} {{site.data.software.Rmodule}}
```

The example above will import version {{site.data.software.Rversion}} of R into your environment. This can be verified with:

```shell
{{site.data.terminal.prompt}} R --version
R version 4.3.3 (2024-02-29) -- "Angel Food Cake"
Copyright (C) 2024 The R Foundation for Statistical Computing
Platform: x86_64-pc-linux-gnu (64-bit)

R is free software and comes with ABSOLUTELY NO WARRANTY.
You are welcome to redistribute it under the terms of the
GNU General Public License versions 2 or 3.
For more information about these matters see
https://www.gnu.org/licenses/.
```
The R environment modules are preconfigured with a large number of installed packages (MCMC, ggplot2, Tidyverse etc). To query the full list of installed packages:

```shell
{{site.data.terminal.prompt}} R -e 'installed.packages()'
```

Many of these packages involve use of Python libraries, and hence loading an R environment module automatically loads a compatible version of Python. 

```shell
{{site.data.terminal.prompt}} module purge
{{site.data.terminal.prompt}} python --version
-bash: python: command not found
{{site.data.terminal.prompt}} module {{site.data.software.Rtoolchain}} {{site.data.software.Rmodule}}
{{site.data.terminal.prompt}} python --version 
Python {{site.data.software.defaultpyver}}
```

## Installing additional R packages from CRAN

Before installing additional R packages first check if your requirements are met by the packages already provided by the R environment module you are using as per above.

Installing some R packages can involve long compilations and so should be done within interactive sessions on compute nodes rather than the login node. First request an interactive session on a compute node, e.g. 16 cores for 2 hours.

```shell
{{site.data.terminal.prompt}} salloc --account=suxxx-somebudget -N 1 -n 16 --mem-per-cpu=3850 --time=2:00:00
```

Once the requested resource is available the command prompt will change to indicate that subsequent commands will be executed within on the allocated compute node.

```shell
[user@node001(sulis) ~]$ 
```

One can then start an interactive R session to install packages.

```
[user@node001(sulis) R]$ R

R version 4.3.3 (2024-02-29) -- "Angel Food Cake"
Copyright (C) 2024 The R Foundation for Statistical Computing
Platform: x86_64-pc-linux-gnu (64-bit)

R is free software and comes with ABSOLUTELY NO WARRANTY.
You are welcome to redistribute it under certain conditions.
Type 'license()' or 'licence()' for distribution details.

  Natural language support but running in an English locale

R is a collaborative project with many contributors.
Type 'contributors()' for more information and
'citation()' on how to cite R or R packages in publications.

Type 'demo()' for some demos, 'help()' for on-line help, or
'help.start()' for an HTML browser interface to help.
Type 'q()' to quit R.

> 
```
For example to install the [RSpectra](https://cran.r-project.org/web/packages/RSpectra/index.html) package from CRAN, and its necessary dependencies:

```R
> install.packages("RSpectra")
```
Users do not have the necessary file system permissions to install or update R packages system-wide, and so will be prompted to accept a default user library location in their home directory. This is recommended for most users. 

When prompted to select a CRAN mirror from which to download the package, select option 0 to choose a local mirror automatically.

Following the lengthy compilation of all dependencies we can import this newly installed library and run one of the example scripts installed along with it.

```R
> library("RSpectra")
> source(system.file("examples/eigs.R", package="RSpectra"))
```

## Bioconductor packages

The [Bioconductor project](http://www.bioconductor.org/) provides a curated set of R packages for bioinformatics which are often closer to the bleeding-edge version number than those included in the latest R release. 

<!--- The installation of R can be extended to include the Bioconductor core package by loading an appropriate environment module. Searching via
```bash
{{site.data.terminal.prompt}} module spider R-bundle-Bioconductor
```
will provide a list of environment modules for the installed versions which can be queried for loading instructions. For example:

```bash
{{site.data.terminal.prompt}} module load GCC/11.2.0 OpenMPI/4.1.1 R-bundle-Bioconductor/3.14-R-4.1.2
``` -->

Users who wish to manage their own set of Bioconductor packages from scratch can do so by following the instructions above to install the `BiocManager` R package. For example, inside an interactive R session running on a compute node:

```R
> install.packages("BiocManager")
```

Bioconductor packages can then be installed with only minimal modification to the installation instructions on the Bioconductor website. For example to install the `GenomicFeatures` package:

```R
> BiocManager::install("GenomicFeatures")
```

This will install the requested package. An informational message will be displayed stating that packages installed in the system-wide package library could not be upgraded. This can be ignored. The latest versions will be installed into the default location in your home directory instead.

Use the `packageVersion()` function to check which version of a particular package is in use. For example, if using the same `{{site.data.software.Rmodule}}` module as above we can update the `broom` package to the version currently used by Bioconductor.

```R
> packageVersion("broom")
[1] '0.7.10'
> BiocManager::install("broom")
packageVersion("broom")
[1] '0.7.12'
```

## Running R in batch mode

As above, R can be used inside interactive sessions by requesting an appropriate session on a compute node using `salloc`. This method is primarily appropriate for debugging/testing of R scripts and installing packages. Long running computational workloads should be submitted to the batch system using `sbatch` to run when the requested resources become available.

R has excellent support for running scripts in batch mode. An example SLURM job submission script for a single process R calculation is shown below.

<p class="codeblock-label">Rserial.slurm</p>
```bash
#!/bin/bash
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=1
#SBATCH --mem-per-cpu={{site.data.slurm.cnode_ram_per_core}}
#SBATCH --time=08:00:00
#SBATCH --account=suxxx-somebudget

module purge
module load {{site.data.software.Rtoolchain}} {{site.data.software.Rmodule}}

srun R CMD BATCH "--args <my_args>" my_script.R
```

Where `<my_args>` should be replace with any command line arguments read by the script `my_script.R` via use of the R function `commandArgs(trailingOnly = TRUE)`. On execution, output normally printed to the console will be written to the file `my_script.Rout` and any plots generated by the script will be combined into `Rplots.pdf`. 

Using command line arguments to pass input into R scripts can be useful when constructing large parallel jobs which consist of a single R script run for many different inputs, for example using [job arrays](../../advanced/ensemble/jobarrays) or creating bash functions which launch R scripts for use with [GNU Parallel](../../advanced/ensemble/gnuparallel).

## Parallelism in R

Parallelism can be implemented within R scripts to accelerate computation. Two methods are:

- Single node parallelism using `mclapply` from the `parallel` package, or using parallel `foreach` from the `doParallel` package. See the [Single node jobs](../batchq/singlenode) for an example job submission script using `mclapply` and the [doParallel documentation](https://cran.r-project.org/web/packages/doParallel/vignettes/gettingstartedParallel.pdf) for examples using that method.

- Parallelism which can span multiple nodes by creating a "cluster" object within R that can be used by functions such as `parLapply`. Within a SLURM environment, the most convenient way to create this is via an MPI cluster type. An example job submission script is given on the [MPI jobs](../../gettingstarted/batchq/mpi) page.

Many R calculations will use packages that implement these or similar methods of parallelism within functions, such that parallelism is "hidden" from the end user. Users are encouraged to consult the documentation for the packages they use and seek help from the appropriate [support contact](../../support) in constructing an appropriate job submission script. 