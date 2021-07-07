---
layout: page
title: Python 
parent: Software
grand_parent: Getting Started
nav_order: 3
---

# Python 
{: .no_toc }

1. TOC
{:toc}

## Python environment modules

Sulis runs the CentOS 8 operating system, which includes Python 3.6. Many users will instead prefer to use newer versions of Python provided via the environment module system. Scientific packages provided via the module system will be built and configured for these newer versions and not the default CentOS Python installation.

Use `module spider Python` to query the available Python builds, and then load using (for example):

```shell
{{site.data.terminal.prompt}} module load {{site.data.software.defaultgcccore}} {{site.data.software.defaultpython}}
```

Invoking `python` will now utilise the newer version.

```shell
{{site.data.terminal.prompt}} python --version
{{site.data.software.defaultpyver}}
```

## Common Python packages

Most scientific applications will make use of standard Python packages such as NumPy, SciPy, Pandas etc. A "bundle" of common packages can be imported into your environment via the SciPy-bundle module. These have been installed using the optimal build
settings for the Sulis hardware. There should be no need for users to install their own versions of these packages. 

In many cases it may be sensible to search and load a SciPy module rather than a Python module directly. The appropriate version of Python will be loaded automatically as a prerequisite.

```shell
{{site.data.terminal.prompt}} module spider SciPy-bundle
```
Will provide a list of compiler and MPI modules (and perhaps a CUDA module in the case of GPU accelerated Python packages) that must be loaded as a prerequisites. For example
```shell
{{site.data.terminal.prompt}} module load {{site.data.software.defaultfoss}} {{site.data.software.defaultscipy}}
```
will load the {{site.data.software.defaultscipy}} module and its prerequisites (including the necessary version of Python). With the SciPy-bundle module loaded one can query the available Python version and the packages available.
```shell
{{site.data.terminal.prompt}} python --version
{{site.data.software.defaultpyver}}
{{site.data.terminal.prompt}} pip list
<long list of available packages>
```

## Accessing additional packages

Many other Python packages are available via the environment module system. Please [search the software already available](modules/#searching-modules) via the module system before attempting to install additional packages. 

If the package you need is likely to be useful for multiple users at your site or elsewhere then consider requesting a centrally installed build via your local [research computing support team](../../support) (HPC Midlands+ consortium members) or by raising an issue in the [sulis-hpc/support](https://github.com/sulis-hpc/support/issues) repository on GitHub (EPSRC national access users).

For other packages, users can invoke `pip` with the `--user` argument to install packages from pypi.org into their home directory. This may be appropriate for packages which do not perform any processor intensive computation and hence optimisation for the Sulis hardware is not a concern. For example, to install the arrow package:

```shell
{{site.data.terminal.prompt}} pip install --user arrow 
```

Do be aware that Python packages are specific to a particular minor version of Python. For example a package installed via pip within a Python 3.7 environment will not be available within a Python 3.8 environment. 

Advanced users may wish to use [Python virtual environments](https://docs.python.org/3/tutorial/venv.html) to manage multiple, possibly conflicting, versions of packages.  

## Anaconda

We do not recommend use of Anaconda for processor intensive scientific applications on Sulis (or on HPC platforms in general). The Anaconda system modifies your default Python environment is ways which may cause problems for optimised builds of packages provided via the module system. Software distributed via Anaconda is also built for compatibility with the largest range of hardware possible, rather than optimised for the latest hardware.

If processor intensive software needed for use on Sulis is only distributed via Anaconda then please first check with your [support contact](../../suport/) who may be able to create an optimised build for the Sulis hardware from the software's source. In some cases the performance difference can be substantial.

If it is genuinely necessary to use Anaconda for a particular project then please be aware that it can be very difficult or impossible to debug compatibility problems with Anaconda-provided software. 

## Compiling Python packages

In some cases users may need to compile their own Python packages from source. This is usually straightforward if the package provides an appropriate setup script and any prerequisite Python packages have been loaded via the environment module system or otherwise. 

A common pitfall occurs when following build/installation guides from a random blog on the internet which assumes all users have root/admin privileges. Be sure to specify that the package should be installed into your home directory when executing the install step, for example:

```shell
{{site.data.terminal.srcprompt}} python setup.py build
{{site.data.terminal.srcprompt}} python setup.py install --user
```

Again, note that packages built/installed from source will only be available within environments that use the same minor Python version as the build environment.

## Batch vs interactive computation 

Python users new to HPC platforms may not be accustomed to running Python-based calculations as non-interactive scripts submitted to a [batch queue](../batchq/).

Sulis is primarily focused on batch computation. Python-based calculations should therefore be prepared as a script which requires no user input beyond command line arguments or input read from a data file. Plots should be saved to file as there is no display connected to the servers which process your script. 

Help converting interactive Python workflows into scripts suitable for batch computation is available via the [HPC for Data Science video lecture series](https://warwick.ac.uk/hpc4ds), the creation of which was funded by the Alan Turing Institute Tools, Practices and Systems programme. 

<iframe width="560" height="315" src="https://www.youtube.com/embed/videoseries?list=PLHRVUtPOez8xPRtcxXlP3msx7Eo2WZ-Pc" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

In some cases though it can be invaluable to interact directly with the Python interpreter as a debugging aid, or to test algorithms within Jupyter notebooks directly on the Sulis hardware. For such situations see the section on [interactive jobs](../batchq/interactive) and the [application notes on Jupyter](../appnotes/jupyter).

