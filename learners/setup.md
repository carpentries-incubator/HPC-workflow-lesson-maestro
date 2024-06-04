---
title: Setup
---

This tutorial assumes you have access to the LC system [Quartz][quartz] (though
many examples will reference the system [Pascal][pascal], which offers an
identical environment).

If you do not have access to LC systems, you will need to install the
prerequisites yourself.

## Prerequisites

* A python3 environment including
  * sys
  * json
  * matplotlib
  * numpy
  * maestrowf
  * amdahl
* the script [`plot_terse_amdahl_results.py`](files/plot_terse_amdahl_results.py)

### If on LC

If you are working on [Quartz][quartz], [Maestro][maestro] is installed to the Python
environment with binaries in `/usr/global/docs/training/janeh/maestro_venv/bin`.
Instructions for how to use these binaries are contained in the lesson.

You will need the python script at
`/usr/global/docs/training/janeh/maestro_venv/plot_terse_amdahl_results.py`
or [directly from GitHub][plot_script].

### If on your own

Some Maestro commands will work locally but others won't make sense unless
you're connected to a cluster with slurm installed. Wherever you're working,
you'll need to install the prerequisite python packages listed above and
download the python plotting script.

## How to connect over ssh

::::::::::::::::::::::::::::::::::::::: discussion

### Details

The application you use to connect to Quartz will depend on the type
of local machine you're working on.

:::::::::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::: spoiler

### Windows

Use [XWin32][xwin] or RealVNC's [VNC Viewer][rvnc].

::::::::::::::::::::::::

:::::::::::::::: spoiler

### MacOS

Use [Terminal.app][tapp] or RealVNC's [VNC Viewer][rvnc].

::::::::::::::::::::::::

:::::::::::::::: spoiler

### Linux

Use [Terminal][term] or RealVNC's [VNC Viewer][rvnc].

::::::::::::::::::::::::

<!-- links -->
[maestro]: https://maestrowf.readthedocs.io/en/latest/
[pascal]: https://hpc.llnl.gov/hardware/compute-platforms/pascal
[plot_script]: https://github.com/carpentries-incubator/hpc-workflows/raw/main/episodes/files/plot_terse_amdahl_results.py
[quartz]: https://hpc.llnl.gov/hardware/compute-platforms/quartz
[rvnc]: https://www.realvnc.com/en/connect/download/viewer/
[tapp]: https://support.apple.com/guide/terminal/welcome/mac
[term]: https://help.ubuntu.com/community/UsingTheTerminal
[xwin]: https://www.starnet.com/xwin32/
