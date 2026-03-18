---
title: Accessing software via Modules
teaching: 30
exercises: 15
---



::::::::::::::::::::::::::::::::::::::: objectives

- Load and use a software package.
- Explain how the shell environment changes when the module mechanism loads or unloads packages.

::::::::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::: questions

- How do we load and unload software packages?

::::::::::::::::::::::::::::::::::::::::::::::::::

On a high-performance computing system, it is seldom the case that the software
we want to use is available when we log in. It is installed, but we will need
to "load" it before it can run.

Before we start using individual software packages, however, we should
understand the reasoning behind this approach. The three biggest factors are:

- software incompatibilities
- versioning
- dependencies

Software incompatibility is a major headache for programmers. Sometimes the
presence (or absence) of a software package will break others that depend on
it. Two well known examples are Python and C compiler versions.
Python 3 famously provides a `python` command that conflicts with that provided
by Python 2. Software compiled against a newer version of the C libraries and
then run on a machine that has older C libraries installed will result in an
opaque `'GLIBCXX_3.4.20' not found` error.

Software versioning is another common issue. A team might depend on a certain
package version for their research project -- if the software version was to
change (for instance, if a package was updated), it might affect their results.
Having access to multiple software versions allows a set of researchers to
prevent software versioning issues from affecting their results.

Dependencies are where a particular software package (or even a particular
version) depends on having access to another software package (or even a
particular version of another software package). For example, the VASP
materials science software may require a particular version of the
FFTW (Fastest Fourier Transform in the West) software library available for it
to work.

## Environment Modules

Environment modules are the solution to these problems. A *module* is a
self-contained description of a software package -- it contains the
settings required to run a software package and, usually, encodes required
dependencies on other software packages.

There are a number of different environment module implementations commonly
used on HPC systems: the two most common are *TCL modules* and *Lmod*. Both of
these use similar syntax and the concepts are the same so learning to use one
will allow you to use whichever is installed on the system you are using. In
both implementations the `module` command is used to interact with environment
modules. An additional subcommand is usually added to the command to specify
what you want to do. For a list of subcommands you can use `module -h` or
`module help`. As for all commands, you can access the full help on the *man*
pages with `man module`.

On login you may start out with a default set of modules loaded or you may
start out with an empty environment; this depends on the setup of the system
you are using.

### Listing Available Modules

To see available software modules, use `module avail`:


```bash
[yourUsername@sorgan ~]$ module avail | less
```

```output
----------------------- /export/spack/share/spack/modules/linux-rocky9-x86_64_v3 -----------------------
   abseil-cpp/20240722.0        gdb/15.2                           mpich/4.2.3-ucx
   apptainer/1.3.6              git-filter-repo/2.38.0             neovim/stable
   casacore/3.6.1               graphviz/12.1.0                    neovim/0.10.2         (D)
   ccache/4.10.2                hyperfine/1.18.0                   ninja/1.12.1
   cfitsio/4.5.0                intel-oneapi-vtune/2025.0.1        npm/9.3.1
   cmake/3.31.4          (D)    lazygit/0.44.1                     nvhpc/24.11
   cuda/12.1.1                  libaio/0.3.113                     openblas/0.3.29
   cuda/12.6.3           (D)    libffi/3.4.6                       openmpi/2.1.6-ofi-ucx
   cutlass/3.4.1                liburing/2.3                       python/3.11.2
   ffmpeg/7.1                   llvm/17.0.6                        python/3.13.1         (D)
   gcc/10.4.0                   llvm/19.1.6                 (D)    ucx/1.17.0
   gcc/13.3.0                   mold/2.36.0                        valgrind/3.23.0
   gcc/14.2.0            (D)    moreutils/0.65

-------------------------------------- /opt/ohpc/pub/modulefiles ---------------------------------------
   cmake/3.24.2    gnu14/14.2.0    hwloc/2.11.1    os    pmix/4.2.9

---------------------------------- /scratch/share/rchavez/modulefiles ----------------------------------
   spack/1.0

  Where:
   D:  Default Module

If the avail list is too long consider trying:
```

Note that piping the output through `less` allows us to search within the output using the <kbd>/</kbd> key.

### Listing Currently Loaded Modules

You can use the `module list` command to see which modules you currently have
loaded in your environment. If you have no modules loaded, you will see a
message telling you so.

```bash
[yourUsername@sorgan ~]$ module list
```

```output
No Modulefiles Currently Loaded.
```

## Loading and Unloading Software

To load a software module, use `module load`.

In this example we will use Python 3. Initially, it is not loaded.
We can test this by using the `which` command. `which` looks for
programs the same way that Bash does, so we can use it to tell us
where a particular piece of software is stored.

```bash
[yourUsername@sorgan ~]$ which python3
```


If the `python3` command was unavailable, we would see output like

```output
/usr/bin/which: no python3 in (/home/yourUsername/.local/bin:/export/spack/bin:/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin)
```

Note that this wall of text is really a list, with values separated
by the `:` character. The output is telling us that the `which` command
searched the following directories for `python3`, without success:

```output
/home/yourUsername/.local/bin
/export/spack/bin
/usr/local/bin
/usr/bin
/usr/local/sbin
/usr/sbin
```

However, in our case we do have an existing `python3` available so we see

```output
/usr/bin/python3
```

We need a different Python than the system provided one though, so let us load
a module to access it.

We can load the `python3` command with `module load`:


```bash
[yourUsername@sorgan ~]$ module load python
[yourUsername@sorgan ~]$ which python3
```

```output
/export/spack/opt/spack/linux-rocky9-x86_64_v3/gcc-11.5.0/python-3.13.1-l4e6fblxgvpiiokcstye4x7puxi4722q/bin/python3
```

So, what just happened?

To understand the output, first we need to understand the nature of the `$PATH`
environment variable. `$PATH` is a special environment variable that controls
where a UNIX system looks for software. Specifically `$PATH` is a list of
directories (separated by `:`) that the OS searches through for a command
before giving up and telling us it can't find it. As with all environment
variables we can print it out using `echo`.

```bash
[yourUsername@sorgan ~]$ echo $PATH
```

```output
/export/spack/opt/spack/linux-rocky9-x86_64_v3/gcc-11.5.0/python-3.13.1-l4e6fblxgvpiiokcstye4x7puxi4722q/bin:/home/yourUsername/.local/bin:/export/spack/bin:/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin
```

You'll notice a similarity to the output of the `which` command. In this case,
there's only one difference: the different directory at the beginning. When we
ran the `module load` command, it added a directory to the beginning of our
`$PATH` -- or "prepended to PATH". Let's examine what's there:


```bash
[yourUsername@sorgan ~]$ ls /export/spack/opt/spack/linux-rocky9-x86_64_v3/gcc-11.5.0/python-3.13.1-l4e6fblxgvpiiokcstye4x7puxi4722q/bin
```

```output
idle3     pydoc3     python   python3.13         python3.13-gdb.py  python-config
idle3.13  pydoc3.13  python3  python3.13-config  python3-config
```

Taking this to its conclusion, `module load` will add software to your `$PATH`.
It "loads" software. A special note on this - depending on which version of the
`module` program that is installed at your site, `module load` will also load
required software dependencies.


To demonstrate, let's use `module list`. `module list` shows all loaded
software modules.

```bash
[yourUsername@sorgan ~]$ module list
```

```output
Currently Loaded Modules:
  1) libffi/3.4.6   2) python/3.13.1
```

```bash
[yourUsername@sorgan ~]$ module load casacore
[yourUsername@sorgan ~]$ module list
```

```output
Currently Loaded Modules:
  1) libffi/3.4.6   2) python/3.13.1   
  3) nvhpc/24.11   4) cfitsio/4.5.0   
  5) casacore/3.6.1
```

So in this case, loading the `casacore` module (a suite of c++ libraries for
 radio astronomy data processing), also loaded `nvhpc/24.11` and
`cfitsio/4.5.0` as well. Let's try unloading the
`GROMACS` package.

```bash
[yourUsername@sorgan ~]$ module unload casacore
[yourUsername@sorgan ~]$ module list
```

```output
Currently Loaded Modules:
  1) libffi/3.4.6   2) python/3.13.1
```

So using `module unload` "un-loads" a module, and depending on how a site is
configured it may also unload all of the dependencies (in our case it does
not). If we wanted to unload everything at once, we could run `module purge`
(unloads everything).

```bash
[yourUsername@sorgan ~]$ module purge
[yourUsername@sorgan ~]$ module list
```

```output
No modules loaded
```

Note that `module purge` is informative. It will also let us know if a default
set of "sticky" packages cannot be unloaded (and how to actually unload these
if we truly so desired).

Note that this module loading process happens principally through
the manipulation of environment variables like `$PATH`. There
is usually little or no data transfer involved.

The module loading process manipulates other special environment
variables as well, including variables that influence where the
system looks for software libraries, and sometimes variables which
tell commercial software packages where to find license servers.

The module command also restores these shell environment variables
to their previous state when a module is unloaded.

## Software Versioning

So far, we've learned how to load and unload software packages. This is very
useful. However, we have not yet addressed the issue of software versioning. At
some point or other, you will run into issues where only one particular version
of some software will be suitable. Perhaps a key bugfix only happened in a
certain version, or version X broke compatibility with a file format you use.
In either of these example cases, it helps to be very specific about what
software is loaded.

Let's examine the output of `module avail` more closely, using the pager since
there may be reams of output:


```bash
[yourUsername@sorgan ~]$ module avail | less
```

```output
----------------------- /export/spack/share/spack/modules/linux-rocky9-x86_64_v3 -----------------------
   abseil-cpp/20240722.0        gdb/15.2                           mpich/4.2.3-ucx
   apptainer/1.3.6              git-filter-repo/2.38.0             neovim/stable
   casacore/3.6.1               graphviz/12.1.0                    neovim/0.10.2         (D)
   ccache/4.10.2                hyperfine/1.18.0                   ninja/1.12.1
   cfitsio/4.5.0                intel-oneapi-vtune/2025.0.1        npm/9.3.1
   cmake/3.31.4          (D)    lazygit/0.44.1                     nvhpc/24.11
   cuda/12.1.1                  libaio/0.3.113                     openblas/0.3.29
   cuda/12.6.3           (D)    libffi/3.4.6                       openmpi/2.1.6-ofi-ucx
   cutlass/3.4.1                liburing/2.3                       python/3.11.2
   ffmpeg/7.1                   llvm/17.0.6                        python/3.13.1         (D)
   gcc/10.4.0                   llvm/19.1.6                 (D)    ucx/1.17.0
   gcc/13.3.0                   mold/2.36.0                        valgrind/3.23.0
   gcc/14.2.0            (D)    moreutils/0.65

-------------------------------------- /opt/ohpc/pub/modulefiles ---------------------------------------
   cmake/3.24.2    gnu14/14.2.0    hwloc/2.11.1    os    pmix/4.2.9

---------------------------------- /scratch/share/rchavez/modulefiles ----------------------------------
   spack/1.0

  Where:
   D:  Default Module

If the avail list is too long consider trying:
```

If the software your Slurm script runs requires on a specific version
of a dependency, make sure you use the full name of the module, rather
than the _default_ loaded when you give only its name (up to the first
slash).

:::::::::::::::::::::::::::::::::::::::  challenge

## Using Software Modules in Scripts

Create a job that is able to run `python3 --version`. Remember, no software
is loaded by default! Running a job is just like logging on to the system
(you should not assume a module loaded on the login node is loaded on a
compute node).

:::::::::::::::  solution

## Solution

```bash
[yourUsername@sorgan ~]$ nano python-module.sh
[yourUsername@sorgan ~]$ cat python-module.sh
```

```output
#!/bin/bash

#SBATCH -p cpu
#SBATCH -t 00:00:30

module load python

python3 --version
```

```bash
[yourUsername@sorgan ~]$ sbatch  python-module.sh
```

:::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::::::


:::::::::::::::::::::::::::::::::::::::: keypoints

- Load software with `module load softwareName`.
- Unload software with `module unload`
- The module system handles software versioning and package conflicts for you automatically.

::::::::::::::::::::::::::::::::::::::::::::::::::
