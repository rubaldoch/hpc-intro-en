---
title: Running a parallel job
teaching: 30
exercises: 60
---



::::::::::::::::::::::::::::::::::::::: objectives

- Install a Python package using `pip`
- Prepare a job submission script for the parallel executable.
- Launch jobs with parallel execution.
- Record and summarize the timing and accuracy of jobs.
- Describe the relationship between job parallelism and performance.

::::::::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::: questions

- How do we execute a task in parallel?
- What benefits arise from parallel execution?
- What are the limits of gains from execution in parallel?

::::::::::::::::::::::::::::::::::::::::::::::::::

We now have the tools we need to run a multi-processor job. This is a very
important aspect of HPC systems, as parallelism is one of the primary tools
we have to improve the performance of computational tasks.

If you disconnected, log back in to the cluster.

```bash
[you@laptop:~]$ ssh -p 6868 yourUsername@ssh.lsc.ic.unicamp.br
[yourUsername@ssh ~]$ ssh sorgan
```

## Install the Amdahl Program

With the Amdahl source code on the cluster, we can install it to gain access to the `amdahl` executable. Since the Amdahl program is written in Python, we need to load a Python module and create a virtual environment for the installation.

First, load the Python module, create a virtual environment, and activate it. Then, move into the extracted directory, and use the Package Installer for Python(`pip`) to install the program within the environment:

```bash
[yourUsername@sorgan ~]$ module load 
[yourUsername@sorgan ~]$ python3 -m venv venv
[yourUsername@sorgan ~]$ source venv/bin/activate
(venv) [yourUsername@sorgan] ~$ cd amdahl
(venv) [yourUsername@sorgan] ~$ pip install .
```

### MPI for Python

The Amdahl code has one dependency: **mpi4py**.
If it hasn't already been installed on the cluster, `pip` will attempt to
collect mpi4py from the Internet and install it for you.
If this fails due to a one-way firewall, you must retrieve mpi4py on your
local machine and upload it, just as we did for Amdahl.

::::::::::::::::::::::::::::::::::::::  discussion

## Retrieve and Upload `mpi4py`

If installing Amdahl failed because mpi4py could not be installed,
retrieve the tarball from <https://github.com/mpi4py/mpi4py/tarball/master>
then `rsync` it to the cluster, extract, and install:

```bash
[you@laptop:~]$ wget -O mpi4py.tar.gz https://github.com/mpi4py/mpi4py/releases/download/3.1.4/mpi4py-3.1.4.tar.gz
[you@laptop:~]$ scp -p 6868 mpi4py.tar.gz yourUsername@ssh.lsc.ic.unicamp.br:
# or
[you@laptop:~]$ rsync --port=6868 -avP mpi4py.tar.gz yourUsername@ssh.lsc.ic.unicamp.br:
```

```bash
[you@laptop:~]$ ssh -p 6868 yourUsername@ssh.lsc.ic.unicamp.br
[yourUsername@sorgan ~]$ tar -xvzf mpi4py.tar.gz  # extract the archive
[yourUsername@sorgan ~]$ mv mpi4py* mpi4py        # rename the directory
[yourUsername@sorgan ~]$ source venv/bin/activate # activate the virtual environment
(venv) [yourUsername@sorgan] ~$ cd mpi4py
(venv) [yourUsername@sorgan] ~$ pip install .
(venv) [yourUsername@sorgan] ~$ cd ../amdahl
(venv) [yourUsername@sorgan] ~$ pip install .
```

::::::::::::::::::::::::::::::::::::::::::::::::::

## Help!

Many command-line programs include a "help" message. Try it with `amdahl`:

```bash
(venv) [yourUsername@sorgan] ~$ amdahl --help
```

```error
Traceback (most recent call last):
  File "/home/yourUsername/venv/bin/amdahl", line 5, in <module>
    from amdahl.amdahl import amdahl
  File "/home/yourUsername/venv/lib/python3.13/site-packages/amdahl/__init__.py", line 1, in <module>
    from . import amdahl
  File "/home/yourUsername/venv/lib/python3.13/site-packages/amdahl/amdahl.py", line 7, in <module>
    from mpi4py import MPI
  File "<frozen importlib._bootstrap>", line 1360, in _find_and_load
  File "<frozen importlib._bootstrap>", line 1322, in _find_and_load_unlocked
  File "<frozen importlib._bootstrap>", line 1262, in _find_spec
  File "/home/yourUsername/venv/lib/python3.13/site-packages/mpi4py/_mpiabi.py", line 290, in find_spec
    mpiabi_suffix = _get_mpiabi_suffix(fullname)
  File "/home/yourUsername/venv/lib/python3.13/site-packages/mpi4py/_mpiabi.py", line 276, in _get_mpiabi_suffix
    mpiabi = _get_mpiabi()
  File "/home/yourUsername/venv/lib/python3.13/site-packages/mpi4py/_mpiabi.py", line 258, in _get_mpiabi
    mpiabi = _get_mpiabi_from_libmpi(libmpi)
  File "/home/yourUsername/venv/lib/python3.13/site-packages/mpi4py/_mpiabi.py", line 217, in _get_mpiabi_from_libmpi
    lib = _dlopen_libmpi(libmpi)
  File "/home/yourUsername/venv/lib/python3.13/site-packages/mpi4py/_mpiabi.py", line 210, in _dlopen_libmpi
    raise RuntimeError("\n".join(errors))
RuntimeError: cannot load MPI library
/home/yourUsername/venv/lib/libmpi.so: cannot open shared object file: No such file or directory
/home/yourUsername/venv/lib/libmpi.so.12: cannot open shared object file: No such file or directory
/home/yourUsername/venv/lib/libmpi.so.40: cannot open shared object file: No such file or directory
libmpi.so: cannot open shared object file: No such file or directory
libmpi.so.12: cannot open shared object file: No such file or directory
libmpi.so.40: cannot open shared object file: No such file or directory
```

This error message indicates that the `amdahl` program is trying to use the MPI library, but it can't find it. This is a common issue when installing `mpi4py` without having an MPI implementation installed and loaded on the cluster. To fix this, you will need to load the appropriate MPI module on the cluster before running `amdahl`.

```bash
(venv) [yourUsername@sorgan] ~$ module load mpich
(venv) [yourUsername@sorgan] ~$ amdahl --help
```

```output
usage: amdahl [-h] [-p [PARALLEL_PROPORTION]] [-w [WORK_SECONDS]] [-t] [-e] [-j [JITTER_PROPORTION]]

optional arguments:
  -h, --help            show this help message and exit
  -p [PARALLEL_PROPORTION], --parallel-proportion [PARALLEL_PROPORTION]
                        Parallel proportion: a float between 0 and 1
  -w [WORK_SECONDS], --work-seconds [WORK_SECONDS]
                        Total seconds of workload: an integer greater than 0
  -t, --terse           Format output as a machine-readable object for easier analysis
  -e, --exact           Exactly match requested timing by disabling random jitter
  -j [JITTER_PROPORTION], --jitter-proportion [JITTER_PROPORTION]
                        Random jitter: a float between -1 and +1
```

Now, we no longer see the error message. While this message doesn’t tell us much about what the program *does*, 
it does show the important flags we may want to use when launching it.

## Running the Job on a Compute Node

Create a submission file, requesting one task on a single node, then launch it.


```bash
[yourUsername@sorgan ~]$ nano serial-job.sh
[yourUsername@sorgan ~]$ cat serial-job.sh
```

```bash
#!/bin/bash
#SBATCH -J solo-job
#SBATCH -p cpu
#SBATCH -N 1
#SBATCH -n 1

# Load the computing environment we need
module load python
module load mpich

# Activate the virtual environment
source ~/venv/bin/activate

# Execute the task
amdahl
```

```bash
[yourUsername@sorgan ~]$ sbatch serial-job.sh
```

As before, use the Slurm status commands to check whether your job
is running and when it ends:

```bash
[yourUsername@sorgan ~]$ squeue -u yourUsername
```

Use `ls` to locate the output file. The `-t` flag sorts in
reverse-chronological order: newest first. What was the output?

:::::::::::::::  spoiler

## Read the Job Output

The cluster output should be written to a file in the folder you launched the
job from. For example,

```bash
[yourUsername@sorgan ~]$ ls -t
```

```output
slurm-347087.out  serial-job.sh  amdahl  README.md  LICENSE.txt
```

```bash
[yourUsername@sorgan ~]$ cat slurm-347087.out
```

```output
Doing 30.000 seconds of 'work' on 1 processor,
which should take 30.000 seconds with 0.850 parallel proportion of the workload.

  Hello, World! I am process 0 of 1 on sorgan-cpu5. I will do all the serial 'work' for 4.500 seconds.
  Hello, World! I am process 0 of 1 on sorgan-cpu5. I will do parallel 'work' for 25.500 seconds.

Total execution time (according to rank 0): 30.033 seconds
```

:::::::::::::::::::::::::

As we saw before, two of the `amdahl` program flags set the amount of work and
the proportion of that work that is parallel in nature. Based on the output, we
can see that the code uses a default of 30 seconds of work that is 85%
parallel. The program ran for just over 30 seconds in total, and if we run the
numbers, it is true that 15% of it was marked 'serial' and 85% was 'parallel'.

Since we only gave the job one CPU, this job wasn't really parallel: the same
processor performed the 'serial' work for 4.5 seconds, then the 'parallel' part
for 25.5 seconds, and no time was saved. The cluster can do better, if we ask.

## Running the Parallel Job

The `amdahl` program uses the Message Passing Interface (MPI) for parallelism
-- this is a common tool on HPC systems.

:::::::::::::::::::::::::::::::::::::::::  callout

## What is MPI?

The Message Passing Interface is a set of tools which allow multiple tasks
running simultaneously to communicate with each other.
Typically, a single executable is run multiple times, possibly on different
machines, and the MPI tools are used to inform each instance of the
executable about its sibling processes, and which instance it is.
MPI also provides tools to allow communication between instances to
coordinate work, exchange information about elements of the task, or to
transfer data.
An MPI instance typically has its own copy of all the local variables.


::::::::::::::::::::::::::::::::::::::::::::::::::

While MPI-aware executables can generally be run as stand-alone programs, in
order for them to run in parallel they must use an MPI *run-time environment*,
which is a specific implementation of the MPI *standard*.
To activate the MPI environment, the program should be started via a command
such as `mpiexec` (or `mpirun`, or `srun`, etc. depending on the MPI run-time
you need to use), which will ensure that the appropriate run-time support for
parallelism is included.

:::::::::::::::::::::::::::::::::::::::::  callout

## MPI Runtime Arguments

On their own, commands such as `mpiexec` can take many arguments specifying
how many machines will participate in the execution,
and you might need these if you would like to run an MPI program on your
own (for example, on your laptop).
In the context of a queuing system, however, it is frequently the case that
MPI run-time will obtain the necessary parameters from the queuing system,
by examining the environment variables set when the job is launched.

::::::::::::::::::::::::::::::::::::::::::::::::::

Let's modify the job script to request more cores and use the MPI run-time.


```bash
(venv) [yourUsername@sorgan] ~$ cp serial-job.sh parallel-job.sh
(venv) [yourUsername@sorgan] ~$ nano parallel-job.sh
(venv) [yourUsername@sorgan] ~$ cat parallel-job.sh
```

```bash
#!/bin/bash
#SBATCH -J parallel-job
#SBATCH -p cpu
#SBATCH -N 1
#SBATCH -n 4

# Load the computing environment we need
module load python
module load mpich

# Execute the task
mpiexec amdahl
```

Then submit your job. Note that the submission command has not really changed
from how we submitted the serial job: all the parallel settings are in the
batch file rather than the command line.

```bash
(venv) [yourUsername@sorgan] ~$ sbatch parallel-job.sh
```

As before, use the status commands to check when your job runs.

```bash
(venv) [yourUsername@sorgan] ~$ ls -t
```

```output
slurm-347178.out  parallel-job.sh  slurm-347087.out  serial-job.sh  amdahl  README.md  LICENSE.txt
```

```bash
(venv) [yourUsername@sorgan] ~$ cat slurm-347178.out
```

```output
Doing 30.000 seconds of 'work' on 4 processors,
which should take 10.875 seconds with 0.850 parallel proportion of the workload.

  Hello, World! I am process 0 of 4 on sorgan-cpu5. I will do all the serial 'work' for 4.500 seconds.
  Hello, World! I am process 2 of 4 on sorgan-cpu5. I will do parallel 'work' for 6.375 seconds.
  Hello, World! I am process 1 of 4 on sorgan-cpu5. I will do parallel 'work' for 6.375 seconds.
  Hello, World! I am process 3 of 4 on sorgan-cpu5. I will do parallel 'work' for 6.375 seconds.
  Hello, World! I am process 0 of 4 on sorgan-cpu5. I will do parallel 'work' for 6.375 seconds.

Total execution time (according to rank 0): 10.888 seconds
```

:::::::::::::::::::::::::::::::::::::::  challenge

## Is it 4× faster?

The parallel job received 4× more processors than the serial job:
does that mean it finished in ¼ the time?

:::::::::::::::  solution

## Solution

The parallel job did take *less* time: 11 seconds is better than 30!
But it is only a 2.7× improvement, not 4×.

Look at the job output:

- While "process 0" did serial work, processes 1 through 3 did their
  parallel work.
- While process 0 caught up on its parallel work,
  the rest did nothing at all.

Process 0 always has to finish its serial task before it can start on the
parallel work. This sets a lower limit on the amount of time this job will
take, no matter how many cores you throw at it.

This is the basic principle behind [Amdahl's Law][amdahl], which is one way
of predicting improvements in execution time for a **fixed** workload that
can be subdivided and run in parallel to some extent.

:::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::::::

## How Much Does Parallel Execution Improve Performance?

In theory, dividing up a perfectly parallel calculation among *n* MPI processes
should produce a decrease in total run time by a factor of *n*.
As we have just seen, real programs need some time for the MPI processes to
communicate and coordinate, and some types of calculations can't be subdivided:
they only run effectively on a single CPU.

Additionally, if the MPI processes operate on different physical CPUs in the
computer, or across multiple compute nodes, even more time is required for
communication than it takes when all processes operate on a single CPU.

In practice, it's common to evaluate the parallelism of an MPI program by

- running the program across a range of CPU counts,
- recording the execution time on each run,
- comparing each execution time to the time when using a single CPU.

Since "more is better" -- improvement is easier to interpret from increases in
some quantity than decreases -- comparisons are made using the speedup factor
*S*, which is calculated as the single-CPU execution time divided by the multi-CPU
execution time. For a perfectly parallel program, a plot of the speedup *S*
versus the number of CPUs *n* would give a straight line, *S* = *n*.

Let's run one more job, so we can see how close to a straight line our `amdahl`
code gets.


```bash
(venv) [yourUsername@sorgan] ~$ nano parallel-job.sh
(venv) [yourUsername@sorgan] ~$ cat parallel-job.sh
```

```bash
#!/bin/bash
#SBATCH -J parallel-job
#SBATCH -p cpu
#SBATCH -N 1
#SBATCH -n 8

# Load the computing environment we need
module load python
module load mpich

# Execute the task
mpiexec amdahl
```

Then submit your job. Note that the submission command has not really changed
from how we submitted the serial job: all the parallel settings are in the
batch file rather than the command line.

```bash
(venv) [yourUsername@sorgan] ~$ sbatch parallel-job.sh
```

As before, use the status commands to check when your job runs.

```bash
(venv) [yourUsername@sorgan] ~$ ls -t
```

```output
slurm-347271.out  parallel-job.sh  slurm-347178.out  slurm-347087.out  serial-job.sh  amdahl  README.md  LICENSE.txt
```

```bash
(venv) [yourUsername@sorgan] ~$ cat slurm-347178.out
```

```output
which should take 7.688 seconds with 0.850 parallel proportion of the workload.

  Hello, World! I am process 4 of 8 on sorgan-cpu5. I will do parallel 'work' for 3.188 seconds.
  Hello, World! I am process 0 of 8 on sorgan-cpu5. I will do all the serial 'work' for 4.500 seconds.
  Hello, World! I am process 2 of 8 on sorgan-cpu5. I will do parallel 'work' for 3.188 seconds.
  Hello, World! I am process 1 of 8 on sorgan-cpu5. I will do parallel 'work' for 3.188 seconds.
  Hello, World! I am process 3 of 8 on sorgan-cpu5. I will do parallel 'work' for 3.188 seconds.
  Hello, World! I am process 5 of 8 on sorgan-cpu5. I will do parallel 'work' for 3.188 seconds.
  Hello, World! I am process 6 of 8 on sorgan-cpu5. I will do parallel 'work' for 3.188 seconds.
  Hello, World! I am process 7 of 8 on sorgan-cpu5. I will do parallel 'work' for 3.188 seconds.
  Hello, World! I am process 0 of 8 on sorgan-cpu5. I will do parallel 'work' for 3.188 seconds.

Total execution time (according to rank 0): 7.697 seconds
```

::::::::::::::::::::::::::::::::::::::  discussion

## Non-Linear Output

When we ran the job with 4 parallel workers, the serial job wrote its output
first, then the parallel processes wrote their output, with process 0 coming
in first and last.

With 8 workers, this is not the case: since the parallel workers take less
time than the serial work, it is hard to say which process will write its
output first, except that it will *not* be process 0!

::::::::::::::::::::::::::::::::::::::::::::::::::

Now, let's summarize the amount of time it took each job to run:

| Number of CPUs | Runtime (sec) |
| -------------- | ------------- |
| 1              | 30\.033        |
| 4              | 10\.888        |
| 8              | 7\.697         |

Then, use the first row to compute speedups $S$, using Python as a command-line
calculator and the formula

$$
S(t_{n}) = \frac{t_{1}}{t_{n}}
$$

```bash
[yourUsername@sorgan ~]$ for n in 30.033 10.888 7.697; do python3 -c "print(30.033 / $n)"; done
```

| Number of CPUs | Speedup       | Ideal |
| -------------- | ------------- | ----- |
| 1              | 1\.0           | 1     |
| 4              | 2\.75          | 4     |
| 8              | 3\.90          | 8     |

The job output files have been telling us that this program is performing 85%
of its work in parallel, leaving 15% to run in serial. This seems reasonably
high, but our quick study of speedup shows that in order to get a 4× speedup,
we have to use 8 or 9 processors in parallel. In real programs, the speedup
factor is influenced by

- CPU design
- communication network between compute nodes
- MPI library implementations
- details of the MPI program itself

Using Amdahl's Law, you can prove that with this program, it is *impossible*
to reach 8× speedup, no matter how many processors you have on hand. Details of
that analysis, with results to back it up, are left for the next class in the
HPC Carpentry workshop, *HPC Workflows*.

In an HPC environment, we try to reduce the execution time for all types of
jobs, and MPI is an extremely common way to combine dozens, hundreds, or
thousands of CPUs into solving a single problem. To learn more about
parallelization, see the [parallel novice lesson][parallel-novice] lesson.


[amdahl]: https://en.wikipedia.org/wiki/Amdahl\'s_law
[parallel-novice]: https://www.hpc-carpentry.org/hpc-parallel-novice/


:::::::::::::::::::::::::::::::::::::::: keypoints

- Parallel programming allows applications to take advantage of parallel hardware.
- The queuing system facilitates executing parallel tasks.
- Performance improvements from parallel execution do not scale linearly.

::::::::::::::::::::::::::::::::::::::::::::::::::
