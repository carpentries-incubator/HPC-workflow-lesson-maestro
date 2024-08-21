---
title: Setup
---

Tutorial exercises and examples refer to two systems called [Ruby][ruby]
and [Pascal][pascal], which offer identical environments.

If you are working on a different cluster, you will need to install the
prerequisites yourself.

## Prerequisites

* A cluster running Slurm
* A python3 environment including
  * common libraries:
    * sys
    * json
    * matplotlib
    * mpi4py
    * numpy
  * Maestro and related scripts:
    * [amdahl](https://github.com/hpc-carpentry/amdahl)
    * [maestrowf](https://maestrowf.readthedocs.io/en/latest/)
    * [yamllint](https://yamllint.readthedocs.io/en/stable/)
  * the script [`plot_terse_amdahl_results.py`](files/plot_terse_amdahl_results.py)

::::::::::::::::::::::::::::::::::::::: discussion

### Setting up your environment

How you set up your environment with the prerequisites will
depend on where you are running this tutorial.

:::::::::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::: spoiler

### If on your own cluster

Some Maestro commands will work locally but others won't make sense unless
you're connected to a cluster with Slurm installed. Wherever you're working,
you'll need to install the prerequisite Python packages listed above and
download the python plotting script.

The recommended way to install these packages is inside a Python
[virtual environment][venv]. You create one using the `venv` module
and specifying the directory in which your environment will be
contained:

``` bash
mkdir ~/venvs
python3 -m venv ~/venvs/maestro
```

This should create the folder at `~/venvs/maestro`, containing

``` bash
ls ~/venvs/maestro
```

``` output
bin/  include/  lib/  lib64  pyvenv.cfg
```

Now you can activate the environment. If your shell is `bash`,

``` bash
source ~/venvs/maestro/bin/activate
```

Otherwise, use the `activate` command appropriate for your shell.
For example,

``` bash
echo $SHELL
```

``` output
/bin/tcsh
```

``` bash
source ~/venvs/maestro/bin/activate.csh
```

Your terminal prompt should now have a `(maestro)` prefix, indicating
that the environment was successfully activated. This environment
includes `pip`, a Python package manager. Use it to install the
required libraries:

``` bash
pip install amdahl maestrowf yamllint
```

Specifying these three custom libraries should be sufficient:
their dependencies will be pulled in by `pip` automagically!

Finally, copy the `plot_terse_amdahl_results.py` script into
your venv's `bin` directory and make it executable:

``` bash
cd ~/venvs/maestro/bin
wget https://github.com/xorJane/maestro-workflow-lesson/raw/main/episodes/files/plot_terse_amdahl_results.py
chmod +x plot_terse_amdahl_results.py
```


::::::::::::::::::::::::

:::::::::::::::: spoiler

### If on Ruby/Pascal

If you are working on [Ruby][ruby], [Maestro][maestro] is installed to
a Python [virtual environment][venv] with binaries in
`/usr/global/docs/training/janeh/maestro_venv/bin`. Instructions for
how to use these binaries are contained in the lesson.

You will need the python script at
`/usr/global/docs/training/janeh/maestro_venv/plot_terse_amdahl_results.py`
or [directly from GitHub][plot_script].

If you are on LC and trying to set up a virtual environment on your own,
follow the instructions for "If on your own cluster", but add the flag
`--system-site-packages` when creating a virtual environment:
 
``` bash
mkdir ~/venvs
python3 -m venv --system-site-packages ~/venvs/maestro
```

::::::::::::::::::::::::




## How to connect over SSH

::::::::::::::::::::::::::::::::::::::: discussion

### Details

The application you use to connect to your cluster will depend on the
type of local machine you're working on.

:::::::::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::: spoiler

### Windows

We recommend using Git BASH via [git for windows](gitBASH)

::::::::::::::::::::::::

:::::::::::::::: spoiler

### MacOS

Use [Terminal.app][tapp] or similar.

::::::::::::::::::::::::

:::::::::::::::: spoiler

### Linux

Use [Terminal][term] or similar.

::::::::::::::::::::::::

<!-- links -->
[gitBASH]: https://gitforwindows.org/
[maestro]: https://maestrowf.readthedocs.io/en/latest/
[pascal]: https://hpc.llnl.gov/hardware/compute-platforms/pascal
[plot_script]: https://github.com/carpentries-incubator/hpc-workflows/raw/main/episodes/files/plot_terse_amdahl_results.py
[ruby]: https://hpc.llnl.gov/hardware/compute-platforms/ruby
[tapp]: https://support.apple.com/guide/terminal/welcome/mac
[term]: https://help.ubuntu.com/community/UsingTheTerminal
[venv]: https://docs.python.org/3/library/venv.html