---
layout: page
title: Jupyter
parent: Application Notes
nav_order: 4
---
# Jupyter
{: .no_toc }

1. TOC
{:toc}

## Sanity check

Sulis is a service primarily focussed on high throughput batch computing workloads. We not recommend using Sulis for interactive workloads such as those commonly implemented within Jupyter notebooks. This is for the following reasons.

- Resources are allocated via SLURM to maximise utilisation of the hardware. Jobs may not start at a convenient time (e.g. in the the middle of night) to be used interactively. 

- Your account budget will be charged for all resources allocated to a Jupyter job even while no code cells are being executed. Editing and reviewing a notebook served from Sulis can be very wasteful of resources.

We do however recognise that for some purposes (debugging, post processing etc) it can still be desirable to work within a Jupyter notebook on Sulis. This is possible via use of an [interactive session](../gettingstarted/batchq/interactive).

## Launching a notebook server within an interactive session

The follow example requests an interactive session on a Sulis GPU node, reserving all three GPUs and all 128 cores within the node for two hours.

```shell
{{site.data.terminal.prompt}} salloc --account=suxxx-somebudget -p gpu  -N 1 -n 128  --gres=gpu:ampere_a100:3 --mem-per-cpu=3850 --time=2:00:00
```

If a GPU node is available then SLURM should drop you into an interactive session on that node within a few seconds. Otherwise, if the machine is busy you may wish to submit to the `devel` partition (or `gpu-devel` for GPU jobs) which has higher priority but a maximum walltime of one hour. 

Once the interactive session starts the command prompt will change to indicate which GPU node has been allocated, in this case `gpu016`.

```shell
[user@gpu016(sulis) ~]$
```

Make a note of which GPU node you have been allocated.

You can then load appropriate environment modules for your interactive session. Jupyter itself is provisioned via the `IPython` module and you may wish to load additional modules to extend your
 Python environment before launching the notebook server, e.g. Matplotlib, Bokeh etc.

```shell
[user@gpu016(sulis) ~]$ module purge
[user@gpu016(sulis) ~]$ module load {{site.data.software.defaultfosscuda}}
[user@gpu016(sulis) ~]$ module load {{site.data.software.defaultscipy}}
[user@gpu016(sulis) ~]$ module load {{site.data.software.defaultipython}}
 ```

A notebook server can then be launched with:


```shell
[user@gpu016(sulis) ~]$ srun jupyter-notebook --no-browser --ip=`uname -n`
```

This will report a notebook server running on (in this case) `gpu016` with a port number (`8888` in the example below) and an authentication token. 

Copy the URL similar to the following from your terminal to the clipboard for pasting into your
browser at the next step. This will normally be the second of the two URLs offered.

```
http://127.0.0.1:8888/?token=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

## Connecting to the notebook server

This assumes you wish to open the remote Jupyter notebook from the same machine you use to connect to Sulis and are using a terminal-based SSH client. For other configurations (e.g. using a jump host at your institution to connect to the Sulis login node) consult your local research computing support team.

In our example above, the interactive session was provided by the node `gpu016`. We cannot connect 
to this directly, and so must tunnel our web browser traffic. On your local machine/laptop open a new terminal session and 

```shell
[user@mylaptop] ~]$ ssh -J user@login.sulis.ac.uk -N -L 8888:gpu016.sulis.hpc:8888 gpu016.sulis.hpc
```
replacing `gpu016` with the GPU node allocated to your job, and `8888` with the port number reported at the previous step if different from `8888`. Replace `user` with your Sulis username.

This will prompt for the details required to connect to Sulis, and establish an SSH tunnel from port 8888 on your local machine to port 8888 on the specified GPU node. 

You should then be able to paste the URL from the previous step into a web browser and connect to
the notebook server running on Sulis.


## Ending the session

When finished with the Jupyter notebook, return to your original terminal session and terminate the notebook server process with control-C, and then 

```shell
[user@gpu016(sulis) ~]$ exit
```

to terminal the interactive session and return to the login node. This will also terminate the SSH tunnel established in the other session.

