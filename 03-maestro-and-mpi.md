---
title: "MPI applications and Maestro"
teaching: 30
exercises: 20
---

::: objectives

- "Define rules to run parallel applications on the cluster"

:::

::: questions

- "How do I run an MPI application via Maestro on the cluster?"

:::

:::::::::::: callout

If you've opened a new terminal, make sure Maestro is available.

```bash
source /usr/global/docs/training/janeh/maestro_venv/bin/activate
```

:::::::::::::::::

Now it's time to start getting back to our real workflow. We can
execute a command on the cluster, but how do we effectively leverage
cluster resources and actually run in parallel? In this episode, we'll
learn how to execute an application that can be run in parallel.

Our application is called `amdahl` and is available in the python
virtual environment we're already using for `maestro`. Check that you
have access to this binary by running `which amdahl` at the command
line. You should see something like

```bash
which amdahl
```

```output
/usr/global/docs/training/janeh/maestro_venv/bin/amdahl
```

We'll use this binary to see how efficiently we can run a parallel
program on our cluster -- i.e. how the amount of work done per
processor changes as we use more processors.

::: challenge

Without using maestro, simply run amdahl on your login node by typing
`amdahl` on the command line and hitting enter.

```bash
amdahl
```

What is amdahl doing?

::::::solution

You should see output that looks roughly like

```bash
amdahl
```

``` output
Doing 30.000000 seconds of 'work' on 1 processor,
which should take 30.000000 seconds with 0.800000
parallel proportion of the workload.

  Hello, World! I am process 0 of 1 on pascal83.
  I will do all the serial 'work' for 5.243022 seconds.

  Hello, World! I am process 0 of 1 on pascal83.
  I will do parallel 'work' for 25.233023 seconds.

Total execution time (according to rank 0): 30.537750 seconds
```

In short, this program prints the amount of time spent working
serially and the amount of time it spends on the parallel section of a
code. On the login node, only a single task is created, so there
shouldn't be any speedup from running in parallel, but soon we'll use
more tasks to run this program!
::::::
:::

In the last challenge, we saw how the `amdahl` executable behaves when
run on the login node. In the next challenge, let's get `amdahl`
running on the login node using `maestro`.

::: challenge

Using what you learned in episode 1, create a Maestro YAML file that
runs `amdahl` on the login node and captures the output in a file.

::::::solution

Your YAML file might be named `amdahl.yaml` and its contents might
look like

```yml
description:
    name: Amdahl
    description: Run a parallel program

study:
    - name: amdahl
      description: run the amdahl executable
      run:
          cmd: |
               amdahl >> amdahl.out
```

and you would run with `maestro run amdahl.yaml`.

::::::
:::

Next, let's get `amdahl` running in batch, on a compute node.

::: challenge

Update `amdahl.yaml` from the last challenge so that this workflow
runs on a compute node with a single task. Use the examples from
episode 2!

Once you've done this. Examine the output and verify that only a
single task is reporting on its work in your output file.

::::::solution

The contents of `amdahl.yaml` should now look something like

```yml
description:
    name: Amdahl
    description: Run on the cluster

batch:
    type: slurm
    host: quartz # machine to run on
    bank: guests # bank
    queue: pdebug # partition

study:
    - name: amdahl
      description: run on the cluster
      run:
          cmd: |
               amdahl >> amdahl.out
          nodes: 1
          procs: 1
          walltime: "00:00:30"
```

In your `amdahl.out` file, you should see that only a single task --
task 0 of 1 -- is mentioned.

Note -- Exact wording for names and descriptions is not important, but
should help you to remember what this file and its study are doing.
::::::
:::

::: challenge

After checking that `amdahl.yaml` looks similar to the solution above,
update the number of `nodes` and `procs` each to `2` and rerun
`maestro run amdahl.yaml`. How many processes report their work in the
output file?

::::::solution

Your YAML files contents are now updated to

```yml
description:
    name: Amdahl
    description: Run on the cluster

batch:
    type: slurm
    host: quartz # machine to run on
    bank: guests # bank
    queue: pdebug # partition

study:
    - name: amdahl
      description: run on the cluster
      run:
          cmd: |
               amdahl >> amdahl.out
          nodes: 2
          procs: 2
          walltime: "00:00:30"
```

In the study folder, `amdahl.slurm.sh` will look something like

``` bash
#!/bin/bash
#SBATCH --nodes=2
#SBATCH --partition=pdebug
#SBATCH --account=guests
#SBATCH --time=00:01:00
#SBATCH --job-name="amdahl"
#SBATCH --output="amdahl.out"
#SBATCH --error="amdahl.err"
#SBATCH --comment "run Amdahl on the cluster"

amdahl >> amdahl.out
```

and in `amdahl.out`, you probably see something like

```output
Doing 30.000000 seconds of 'work' on 1 processor,
which should take 30.000000 seconds with 0.800000
parallel proportion of the workload.

  Hello, World! I am process 0 of 1 on pascal17.
  I will do all the serial 'work' for 5.324555 seconds.
  
  Hello, World! I am process 0 of 1 on pascal17.
  I will do parallel 'work' for 22.349517 seconds.

Total execution time (according to rank 0): 27.755552 seconds
```

Notice that this output refers to only "1 processor" and mentions
only one process. We requested two processes, but only a single one
reports back! Additionally, we requested two _nodes_, but only one
is mentioned in the above output (`pascal17`).

So what's going on?

If your job were really _using_ both tasks and nodes that were
assigned to it, then both would have written to `amdahl.out`.

The `amdahl` binary is enabled to run in parallel but it's also able
to run in serial. If we want it to run in parallel, we'll have to tell
it so more directly.
::::::
:::

Here's the takeaway from the challenges above: It's not enough to have
both parallel resources and a binary/executable/program that is
enabled to run in parallel. We actually need to invoke MPI in order to
force our parallel program to use parallel resources.

## Maestro and MPI

We didn't really run an MPI application in the last section as we only
ran on one processor. How do we request to run using multiple
processes for a single step?

The answer is that we have to tell Slurm that we want to use MPI. In
the Intro to HPC lesson, the episodes introducing Slurm and running
parallel jobs showed that commands to run in parallel need to use
`srun`. `srun` talks to MPI and allows multiple processors to
coordinate work. A call to `srun` might look something like

```bash
srun -N {# of nodes} -n {number of processes} amdahl >> amdahl.out
```

To make this easier, Maestro offers the shorthand
`$(LAUNCHER)`. Maestro will replace instances of `$(LAUNCHER)` with a
call to `srun`, specifying as many nodes and processes we've already
told Slurm we want to use.

::: challenge

Update `amdahl.yaml` to include `$(LAUNCHER)` in the call to `amdahl`
so that your study's `cmd` field includes

```bash
$(LAUNCHER) amdahl >> amdahl.out
```

Run maestro with the updated YAML and explore the outputs. How many
tasks are mentioned in `amdahl.out`? In the Slurm submission script
created by Maestro (included in the same subdirectory as
`amdahl.out`), what text was used to replace `$(LAUNCHER)`?

:::::: solution

The updated YAML should look something like

```yml
description:
    name: Amdahl
    description: Run a parallel program

batch:
    type: slurm
    host: quartz # machine to run on
    bank: guests # bank
    queue: pdebug # partition

study:
    - name: amdahl
      description: run in parallel
      run:
          # Here's where we include our MPI wrapper:
          cmd: |
               $(LAUNCHER) amdahl >> amdahl.out
          nodes: 2
          procs: 2
          walltime: "00:00:30"
```

Your output file `Amdahl_.../amdahl/amdahl.out` should include "Doing
30.000000 seconds of 'work' on 2 processors" and the submission script
`Amdahl_.../amdahl/amdahl.slurm.sh` should include the line
`srun -n 2 -N 2 amdahl >> amdahl.out`.
Maestro substituted `srun -n 2 -N 2` for `$(LAUNCHER)`!

::::::
:::

::: callout

## Commenting Maestro YAML files

In the solution from the last challenge, the line beginning `#` is a
comment line. Hopefully you are already in the habit of adding
comments to your own scripts. Good comments make any script more
readable, and this is just as true with our YAML files.

:::

## Customizing amdahl output

Another thing about our application `amdahl` is that we ultimately
want to process the output to generate our scaling plot. The output
right now is useful for reading but makes processing harder. `amdahl`
has an option that actually makes this easier for us. To see the
`amdahl` options we can use

```bash
amdahl --help
```

```output
usage: amdahl [-h] [-p [PARALLEL_PROPORTION]] [-w [WORK_SECONDS]] [-t] [-e]

options:
  -h, --help            show this help message and exit
  -p [PARALLEL_PROPORTION], --parallel-proportion [PARALLEL_PROPORTION]
                        Parallel proportion should be a float between 0 and 1
  -w [WORK_SECONDS], --work-seconds [WORK_SECONDS]
                        Total seconds of workload, should be an integer > 0
  -t, --terse           Enable terse output
  -e, --exact           Disable random jitter
```

The option we are looking for is `--terse`, and that will make
`amdahl` print output in a format that is much easier to process,
JSON. JSON format in a file typically uses the file extension `.json`
so let's add that option to our `shell` command _and_ change the file
format of the `output` to match our new command:

```yml
description:
    name: Amdahl
    description: Run a parallel program

batch:
    type: slurm
    host: quartz  # machine to run on
    bank: guests  # bank
    queue: pdebug # partition

study:
    - name: amdahl
      description: run in parallel
      run:
          # Here's where we include our MPI wrapper:
          cmd: |
               $(LAUNCHER) amdahl --terse >> amdahl.json
          nodes: 2
          procs: 2
          walltime: "00:01:30"
```

There was another parameter for `amdahl` that caught my eye. `amdahl` has an
option `--parallel-proportion` (or `-p`) which we might be interested in
changing as it changes the behavior of the code, and therefore has an impact on
the values we get in our results. Let's try specifying a parallel proportion
of 90%:

```yml
description:
    name: Amdahl
    description: Run a parallel program

batch:
    type: slurm
    host: quartz  # machine to run on
    bank: guests  # bank
    queue: pdebug # partition

study:
    - name: amdahl
      description: run in parallel
      run:
          # Here's where we include our MPI wrapper:
          cmd: |
               $(LAUNCHER) amdahl --terse -p .9 >> amdahl.json
          nodes: 2
          procs: 2
          walltime: "00:00:30"
```

::: challenge

Create a YAML file for a value of `-p` of 0.999 (the default value is 0.8)
for the case where we have a single node and 4 parallel processes. Run
this workflow with Maestro to make sure your script is working.

:::::: solution

```yml
description:
    name: Amdahl
    description: Run a parallel program

batch:
    type: slurm
    host: quartz # machine to run on
    bank: guests # bank
    queue: pdebug # partition

study:
    - name: amdahl
      description: run in parallel
      run:
          # Here's where we include our MPI wrapper:
          cmd: |
               $(LAUNCHER) amdahl --terse -p .999 >> amdahl.json
          nodes: 1
          procs: 4
          walltime: "00:00:30"
```

::::::
:::

## Environment variables

Our current directory is probably starting to fill up with directories
starting with `Amdahl_...`, distinguished only by dates and timestamps.
It's probably best to group runs into separate folders to keep things tidy.
One way we can do this is by specifying an `env` section in our YAML
file with a variable called `OUTPUT_PATH` specified in this format:

```yml
env:
    variables:
      OUTPUT_PATH: ./Episode3
```

This `env` block goes above our `study` block; `env` is at the same level of
indentation as `study`. In this case, directories created by runs using this
`OUTPUT_PATH` will all be grouped inside the directory `Episode3`, to help us
group runs by where we are in the lesson.

::: challenge

Modify your YAML so that subsequent runs will be grouped into a shared parent
directory (for example, `Episode3`, as above).

:::::: solution

```yml
description:
    name: Amdahl
    description: Run a parallel program

batch:
    type: slurm
    host: quartz # machine to run on
    bank: guests # bank
    queue: pdebug # partition

env:
    variables:
      OUTPUT_PATH: ./Episode3

study:
    - name: amdahl
      description: run in parallel
      run:
          # Here's where we include our MPI wrapper:
          cmd: |
               $(LAUNCHER) amdahl --terse -p .999 >> amdahl.json
          nodes: 1
          procs: 4
          walltime: "00:00:30"
```

::::::
:::

## Dry-run (`--dry`) mode

It's often useful to run Maestro in `--dry` mode, which causes Maestro to
create scripts and the directory structure without actually running jobs.
You will see this parameter if you run `maestro run --help`.

::: challenge

Do a couple `dry` runs using the script created in the last challenge. This
should help you verify that a new directory "Episode3" gets created for runs
from this episode.

__Note__: `--dry` is an input for `maestro run`, __not__ for `amdahl`. To
do a dry run, you shouldn't need to update your YAML file at all. Instead, you
just run

```bash
maestro run --dry «YAML filename»
```

:::::: solution

After running

```bash
maestro run --dry amdahl.yaml
```

a directory path of the form `Episode3/Amdahl_{DATE}_{TIME}/amdahl` should
be created.

::::::
:::

::: keypoints

- "Adding `$(LAUNCHER)` before commands signals to Maestro to use MPI via `srun`."
- "New Maestro runs can be grouped within a new directory specified by the
  environment variable `OUTPUT_PATH`"
- You can use `--dry` to verify that the expected directory structure and
  scripts are created by a given Maestro YAML file.

:::
