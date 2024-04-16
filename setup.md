---
title: Setup
---

This tutorial assumes you have access to the LC system `Quartz` (though many examples will reference the system `Pascal`, which overs an identical environment).

If you do not have access to LC systems, you will need to install the prerequisites yourself.

## Prerequisites

* A python3 environment including
    * sys
    * json
    * matplotlib
    * numpy
    * maestrowf
    * amdahl
* the script [`plot_terse_amdahl_results.py`](plot_terse_amdahl_results.py)

### If on LC

If you are working on `Quartz`, Maestro is installed to the python environment with binaries in `/usr/global/docs/training/janeh/maestro_venv/bin`. Instructions for how to use these binaries are contained in the lesson.

You will need the python script at `/usr/global/docs/training/janeh/maestro_venv/plot_terse_amdahl_results.py`

### If on your own

Some Maestro commands will work locally but others won't make sense unless you're connected to a cluster with slurm installed. Wherever you're working, you'll need to install the prerequisite python packages listed above and download the python plotting script.

## How to connect over ssh

::::::::::::::::::::::::::::::::::::::: discussion

### Details

The application you use to connect to Quartz will depend on the type of local machine you're working on.

:::::::::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::: spoiler

### Windows

Use XWin32 or RealVNC's VNC Viewer.

::::::::::::::::::::::::

:::::::::::::::: spoiler

### MacOS

Use Terminal.app or RealVNC's VNC Viewer.

::::::::::::::::::::::::


:::::::::::::::: spoiler

### Linux

Use Terminal or RealVNC's VNC Viewer.

::::::::::::::::::::::::
