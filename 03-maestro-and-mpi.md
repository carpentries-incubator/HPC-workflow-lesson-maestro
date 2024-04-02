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

Now it's time to start getting back to our real workflow. We can execute a
command on the cluster, but what about executing the MPI application we are
interested in? Our application is called `amdahl` and is available in the
python virtual environment we're already using for `maestro`. Check that
you have access to this binary by running `which amdahl` at the command
line. You should see something like

```bash
(maestro_venv) janeh@pascal83:~$ which amdahl
```
```output
/usr/global/docs/training/janeh/maestro_venv/bin/amdahl
```

We'll use this binary to see how efficiently we can run a parallel
program on our cluster -- i.e. how the amount of work done per
processor changes as we use more processors.

::: challenge

Copy `hostname.yaml` to a new file, `amdahl.yaml` with 
`cp hostname.yaml amdahl.yaml`.

Open the new file, `amdahl.yaml`. Remove the "hostname-login" step so that
the study in this file has only a single step. Rename the "hostname_batch"
step to "amdahl" or something similar. Update other names and descriptions
in this file to reflect the new binary. Update the command under your study
so that the `amdahl` binary will run and write to an output file.

::::::solution

The contents of `amdahl.yaml` should look something like

```yml
description:
    name: Amdahl
    description: Run a parallel program

batch:
    type: slurm
    host: quartz # machine to run on
    bank: guest # bank
    queue: pbatch # partition

study:
    - name: amdahl
      description: run in parallel
      run:
          cmd: |
               amdahl >> amdahl.out
          nodes: 1
          procs: 1
          walltime: "00:00:30"
```

Exact wording for names and descriptions is not important, but should
help you to remember what this file and its study are doing.

::::::
:::

::: challenge

After checking that `amdahl.yaml` looks similar to the solution above,
run `maestro run amdahl.yaml`. Then, update the number of `nodes` and
`procs` each to `2`. You should also increase the walltime a bit, to
a minute or minute and a half. Then rerun `maestro run amdahl.yaml`.
How does the output change? How did you expect it to change?

*Hint* Remember that if you run `squeue -u <your username>`, you can
see the node(s) assigned to your slurm job.

::::::solution

In your output files (`amdahl.out` if using the script in the last
solution), you probably see output looking something like

```
Doing 30.000000 seconds of 'work' on 1 processor,
 which should take 30.000000 seconds with 0.800000 parallel proportion of the workload.

  Hello, World! I am process 0 of 1 on pascal17. I will do all the serial 'work' for 5.324555 seconds.
  Hello, World! I am process 0 of 1 on pascal17. I will do parallel 'work' for 22.349517 seconds.

Total execution time (according to rank 0): 27.755552 seconds
```

Notice that this output refers to only "1 processor" and mentions
only `pascal17`, even in the job that requested and received two
nodes. You will likely see a different number, but most you will
still see only a single processor mentioned in the output. If you
ran `squeue -u <username>` while the job was in queue, you should
have seen two unique node numbers assigned to your job.

So what's going on? If your job were really *using* both nodes
that were assigned to it, then both processes would have written
to `amdahl.out`.

::::::
:::

## Maestro and MPI

We didn't really run an MPI application in the last section as we only ran on
one processor. How do we request to run on multiple processors for a single
step?

The answer is that we have to tell Slurm that we want to use MPI. In the Intro
to HPC lesson, the episodes introducing Slurm and running parallel jobs showed
that commands to run in parallel need to use `srun`. `srun` talks to MPI and
allows multiple processors to coordinate work. A call to `srun` might look
something like

```bash
srun -N {# of nodes} -n {number of processes} amdahl >> amdahl.out
```

To make this easier, Maestro offers the shorthand `$(LAUNCHER)`. Maestro
will replace instances of `$(LAUNCHER)` with a call to `srun`, specifying
as many nodes and processes we've already told Slurm we want to use.

::: challenge

Update `amdahl.yaml` to include `$(LAUNCHER)` before the call to `amdahl`
in your study's `run` field. Re-run maestro with the updated YAML and
explore the outputs. How many nodes are mentioned in `amdahl.out`?
In the Slurm submission script created by Maestro (included in the same
subdirectory as `amdahl.out`), what text was used to replace `$(LAUNCHER)`?

:::::: solution

The updated YAML should look something like

```yml
description:
    name: Amdahl
    description: Run a parallel program

batch:
    type: slurm
    host: quartz # machine to run on
    bank: guest # bank
    queue: pbatch # partition

study:
    - name: amdahl
      description: run in parallel
      run:
          # Here's where we include our MPI wrapper:
          cmd: |
               $(LAUNCHER) amdahl >> amdahl.out
          nodes: 2
          procs: 2
          walltime: "00:01:30"
```

Your output file `Amdahl_.../amdahl/amdahl.out` should include
"Doing 30.000000 seconds of 'work' on 2 processors" and the submission
script `Amdahl_.../amdahl/amdahl.slurm.sh` should include the line
"srun -n 2 -N 2 amdahl >> amdahl.out". Maestro substituted
`srun -n 2 -N 2` for `$(LAUNCHER)`!

::::::
:::

::: callout
## Commenting Maestro YAML files

In the solution from the last challenge, the line beginning `#` is a comment line. Hopefully 
you are already in the habit of adding comments to your own scripts. Good comments make any
script more readable, and this is just as true with our YAML files.

:::


## Customizing amdahl output

Another thing about our application `amdahl` is that we ultimately want to
process the output to generate our scaling plot. The output right now is useful
for reading but makes processing harder. `amdahl` has an option that actually
makes this easier for us. To see the `amdahl` options we can use

```bash
(maestro_venv) janeh@pascal83:~$ amdahl --help
```
```output
usage: amdahl [-h] [-p [PARALLEL_PROPORTION]] [-w [WORK_SECONDS]] [-t] [-e]

options:
  -h, --help            show this help message and exit
  -p [PARALLEL_PROPORTION], --parallel-proportion [PARALLEL_PROPORTION]
                        Parallel proportion should be a float between 0 and 1
  -w [WORK_SECONDS], --work-seconds [WORK_SECONDS]
                        Total seconds of workload, should be an integer greater than 0
  -t, --terse           Enable terse output
  -e, --exact           Disable random jitter
```
The option we are looking for is `--terse`, and that will make `amdahl` print
output in a format that is much easier to process, JSON. JSON format in a file
typically uses the file extension `.json` so let's add that option to our 
`shell` command _and_ change the file format of the `output` to match our new
command:

```yml
description:
    name: Amdahl
    description: Run a parallel program

batch:
    type: slurm
    host: quartz # machine to run on
    bank: guest # bank
    queue: pbatch # partition

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
changing as it changes the behaviour of the code,and therefore has an impact on
the values we get in our results. Let's try specifying a parallel proportion
of 90%:

```yml
description:
    name: Amdahl
    description: Run a parallel program

batch:
    type: slurm
    host: quartz # machine to run on
    bank: guest # bank
    queue: pbatch # partition

study:
    - name: amdahl
      description: run in parallel
      run:
          # Here's where we include our MPI wrapper:
          cmd: |
               $(LAUNCHER) amdahl --terse -p .9 >> amdahl.json
          nodes: 2
          procs: 2
          walltime: "00:01:30"
```

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

This `env` block goes above our `study` block. In this case, directories
created by runs using this `OUTPUT_PATH` will all be grouped inside the
directory `Episode3`, to help us group runs by where we are in the lesson.


::: challenge

Create a YAML file for a value of `-p` of 0.999 (the default value is 0.8)
for the case where we have a single node and 6 parallel processes. 
Directories for subsequent runs should be grouped into a shared parent
directory (for example, `Episode3`, as above).

:::::: solution

```yml
description:
    name: Amdahl
    description: Run a parallel program

batch:
    type: slurm
    host: quartz # machine to run on
    bank: guest # bank
    queue: pbatch # partition

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
          procs: 6
          walltime: "00:01:30"
```

::::::
:::

## Dry-run (`--dry`) mode

It's often useful to run Maestro in `--dry` mode, which causes Maestro to create scripts
and the directory structure without actually running jobs. You will see this parameter
if you run `maestro run --help`.

::: challenge

Do a dry-run using the script created in the last challenge. This should help you
verify that a new directory gets created for runs from this episode.

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
- "New Maestro runs can be grouped within a new directory specified by the environment
variable `OUTPUT_PATH`"
- You can use `--dry` to verify that the expected directory structure and scripts
are created by a given Maestro YAML file.

:::
