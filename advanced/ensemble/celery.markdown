---
layout: page
title: Celery
parent: Ensemble jobs
grand_parent: Advanced Topics
nav_order: 2
---

# Celery
{: .no_toc }

1. TOC
{:toc}

Celery is a distributed task queue library primarily used as part of web server clusters where it is used as part of large production systems such as Instagram. Celery works by setting up a set of worker processes and a message broker program (effectively a database, usually Redis or RabbitMQ). When a client wants work done it connects to the message broker and adds the new task to the message queue. When a worker process becomes free it connects to the message broker and pulls the next task from the queue. After finishing, the worker process returns the result of the task using the message broker and the client that started the task can recover the result from the message broker.

Celery is based on network technologies that are common in the web server space, not the HPC space so it is not recommended for use on Sulis. Generally it is possible to use other tools intended for HPC instead of Celery. This document first goes through using the more common HPC technologies instead of Celery and then describes how to make Celery work on Sulis

## Alternatives to Celery

### GNU Parallel

If your problem is a series of individual unrelated tasks then you can switch from Celery to GNU parallel. GNU parallel is a general purpose command-line tool for launching many concurrent instances of a program to work through a given list of inputs. It is synchronous (i.e. when a given task is running GNU parallel waits for that task to finish) but distributed (i.e. it uses multiple processors over multiple machines to complete multiple tasks as once) and tasks cannot have interdependencies (i.e. you cannot use the result of one task to determine whether to run another task - all tasks always run). This means that GNU Parallel is only suitable for running a subset of Celery tasks, but it is easier to use than Celery for those tasks. There is a separate page in these docs detailing how to use GNU Parallel, and the GNU parallel documentation is very good so here we will detail how to adapt a Celery script to use GNU parallel

GNU Parallel works by running a single program with different command line parameters, so you will have to adjust your code to work like this. If you are using Celery with a single file containing the functions that you are distributing then it can be converted simply. Consider the file 

<p class="codeblock-label">tasks.py</p>
```python
from celery import Celery

broker = 'redis://localhost:6379/0'
app = Celery('tasks', broker=broker)

@app.task
def add(x, y):
    return x + y

@app.task
def sub(x, y):
    return x - y

``` 

This file is used both by the Celery workers and by the caller script submitting the task. First, you want to remove all of the Celery specific code, leaving you with

<p class="codeblock-label">tasks_gnu.py</p>
```python
def add(x, y):
    return x + y

def sub(x, y):
    return x - y

```

Then you want to add the code that takes the command line parameters. Python has two ways of getting command line parameters "sys.argv" which is very similar to the way that command line parameters are handled in C and C++ code and "getopt" which helps with parsing named command line arguments but is more complex. Both are well documented so here we will concentrate on the simpler "sys.argv" system.

<p class="codeblock-label">tasks_gnu.py</p>
```python
#!/bin/env python3
import sys
def add(x, y):
    return x + y

def sub(x, y):
    return x - y

if (int(sys.argv[1]) == 1):
    print(add(int(sys.argv[2]),int(sys.argv[3])))
else:
    print(sub(int(sys.argv[2]),int(sys.argv[3])))

```

The first line of this script is called a "shebang" line and it allows the script to run from the command line without having to manually invoke the Python interpreter. In this case it will run this script using Python3.

"sys.argv" is a list of all of the command line parameters that are passed to the script. sys.argv[0] is always the name of the script and you can tell how many parameters were passed to the script using len(sys.argv) (note that this is different to C and C++ which have a separate "argc" parameter that says how many arguments there are). This script then converts the first argument into an integer and uses it to determine whether to add or subtract the next two parameters from each other.

Imagine now that you want to run this script so that it adds the numbers from 1-5 inclusive to 10-14 inclusive. Using GNU Parallel you could run this using the command

```bash
parallel --link ./test.py 1 {1} {2} ::: {1..5} ::: {10..14}
```
This isn't a trivial GNU parallel script but it does show the power of the approach. The command (from left to right reads)

* parallel - The GNU parallel program
* --link - Take inputs pairwise rather than take the Cartesian product
* ./test.py - My script to run in parallel
* 1 - Parameter to my script, I want to run the addition code
* {1} - Replace this parameter to my script by the first GNU parallel parameter
* {2} - Replace this parameter to my script by the second GNU parallel parameter
* ::: - This separates GNU parallel parameters from parameters to my script
* {1..5} - The numbers from 1 to 5 inclusive
* ::: - Separate the second GNU parallel parameter from the first
* {10..14} - The numbers from 10 to 15 inclusive

The output of this code is

```bash
11
13
15
17
19
```

and in this case the outputs are in the order of the inputs, but this will not in general be true. NB that simply running *parallel* on the head node of the cluster will not correctly make use of the compute nodes - you should run GNU parallel as detailed in the documentation for GNU parallel.

### Dask

Dask is a large library that covers a wide variety of parallel tasks in scientific programming from distributed data analytics to distributed machine learning, but here we are interested in Dask's implementation of *futures*. Futures are a generic concept in computer programming (in fact Python has futures built in which work in the same way but only work on a single node, not across multiple nodes) which is a function that is started now and runs in the background but is going to give its answer in the future (hence the name). These are basically the same idea as Celery's tasks and are used in much the same way.

## Using Celery

To make Celery work on Sulis you have to write the submission script so that it will 

1. Start the Redis backend server (Redis is easier here than RabbitMQ)
2. Start as many Celery workers as requested
3. Start your core script that distributes tasks to the workers

An example script that shows this is

```bash
#!/bin/bash
#SBATCH --account=suxx-someone
#SBATCH --time=0:05:00
#SBATCH --job-name=celery_test
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=128
#SBATCH --cpus-per-task=1
#SBATCH --mem-per-cpu=3700mb

module purge;module load GCCcore/11.2.0 Python/3.9.6

export IP=`ip a s ib0 | awk '/inet / {print $2}' | cut -d/ -f1`
redis-server --protected-mode no --bind $IP&
srun celery -Atasks worker&
python3 tasks.py

```

The SLURM SBATCH lines are unremarkable and are documented in the general Sulis documentation. The first line of the script proper then gets the IP address of the node that the task is running on. The next line then starts a redis server to act as the message broker without any security but bound only to the local IP address. If you are working with sensitive data (for example holding personal or medical information) then you can easily add password authentication by following the documentation for redis. The penultimate line starts the celery workers running in parallel. With these SBATCH settings 128 workers will be created, each of which has 1 CPU. The final line then runs the Python script that serves the work packages to the celery workers.
