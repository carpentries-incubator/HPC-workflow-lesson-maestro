---
title: "Placeholders"
teaching: 40
exercises: 30
---

::: questions

- "How do I make a generic rule?"

:::

::: objectives

- "Learn to use variables as placeholders"
- "Learn to run many similar Maestro runs at once"

:::

:::::::::::: callout

If you've opened a new terminal, make sure Maestro is available.

```bash
source /usr/global/docs/training/janeh/maestro_venv/bin/activate
```

:::::::::::::::::

## D.R.Y. (Don't Repeat Yourself)

::: callout

In many programming languages, the bulk of the language features are
there to allow the programmer to describe long-winded computational
routines as short, expressive, beautiful code.  Features in Python, R,
or Java, such as user-defined variables and functions are useful in
part because they mean we don't have to write out (or think about) all
of the details over and over again.  This good habit of writing things
out only once is known as the "Don't Repeat Yourself" principle or
D.R.Y.

:::

Maestro YAML files are a form of code and, in any code, repetition can
lead to problems (e.g. we rename a data file in one part of the YAML
but forget to rename it elsewhere).

In this episode, we'll set ourselves up with ways to avoid repeating
ourselves by using _environment variables_ as _placeholders_.

## Placeholders

Over the course of this lesson, we want to use the `amdahl` binary to
show how the execution time of a program changes with the number of
processes used. In on our current setup, to run amdahl for multiple
values of `procs`, we would need to run our workflow, change `procs`,
rerun, and so forth. We'd be repeating our workflow a lot, so let's
first try fixing that by defining multiple rules.

At the end of our last episode, our `amdahl.yaml` file contained the
sections

```yml
(...)

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

Let's call our existing step `amdahl-1` (`name` under `study`) and
create a second step called `amdahl-2` which is exactly the same,
except that it will define `procs: 8`. While we're at it, let's update
`OUTPUT_PATH` so that it is `./Episode4`.

The updated part of the script now looks like

```yml
(...)

env:
    variables:
      OUTPUT_PATH: ./Episode4

study:
    - name: amdahl-1
      description: run in parallel
      run:
          cmd: |
               $(LAUNCHER) amdahl --terse -p .999 >> amdahl.json
          nodes: 1
          procs: 4
          walltime: "00:00:30"
    - name: amdahl-2
      description: run in parallel
      run:
          cmd: |
               $(LAUNCHER) amdahl --terse -p .999 >> amdahl.json
          nodes: 1
          procs: 8
          walltime: "00:00:30"
```

::: challenge

Update `amdahl.yaml` to include the new info shown above. Run a dry
run to see what your output directory structure looks like.

:::

Now let's start to get rid of some of the redundancy in our new
workflow.

First off, defining the parallel proportion (`-p .999`) in two places
makes our lives harder. Now if we want to change this value, we have
to update it in two places, but we can make this easier by using an
environment variable.

Let's create another environment variable in the `variables` second
under `env`. We can define a new parallel proportion as `P:
.999`. Then, under `run`'s `cmd` for each step, we can call this
environment variable with the syntax `$(P)`. `$(P)` holds the place of
and will be substituted by `.999` when Maestro creates a Slurm
submission script for us.

Let's also create an environment variable for our output file,
`amdahl.json` called `OUTPUT` and then call that variable from our
`cmd` fields.

Our updated section will now look like this:

```yml
(...)

env:
    variables:
      P: .999
      OUTPUT_PATH: ./Episode4
      OUTPUT: amdahl.json

study:
    - name: amdahl-1
      description: run in parallel
      run:
          cmd: |
               $(LAUNCHER) amdahl --terse -p $(P) >> $(OUTPUT)
          nodes: 1
          procs: 4
          walltime: "00:00:30"
    - name: amdahl-2
      description: run in parallel
      run:
          cmd: |
               $(LAUNCHER) amdahl --terse -p $(P) >> $(OUTPUT)
          nodes: 1
          procs: 8
          walltime: "00:00:30"
```

We've added two new placeholders to make our YAML script to make it a
tad bit more efficient. Note that we had already been using a
placeholder given to us by Maestro: $(LAUNCHER) holds the place of a
call to `srun <insert resource requests>`.

::: challenge

Run your updated `amdahl.yaml` and check results, to verify your
workflow is working with the changes you've made so far.

::::::solution

The full YAML text is

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
      P: .999
      OUTPUT_PATH: ./Episode4
      OUTPUT: amdahl.json

study:
    - name: amdahl-1
      description: run in parallel
      run:
          cmd: |
               $(LAUNCHER) amdahl --terse -p $(P) >> $(OUTPUT)
          nodes: 1
          procs: 4
          walltime: "00:00:30"
    - name: amdahl-2
      description: run in parallel
      run:
          cmd: |
               $(LAUNCHER) amdahl --terse -p $(P) >> $(OUTPUT)
          nodes: 1
          procs: 8
          walltime: "00:00:30"
```

::::::
:::

## Maestro's global.parameters

We're almost ready to perform our scaling study -- to see how the
execution time changes as we use more processors in the
job. Unfortunately, we're still repeating ourselves a lot because, in
spite of the environment variables we created, most of the information
defined for steps `amdahl-1` and `amdahl-2` is the same. Only the
`procs` field changes!

A great way to avoid repeating ourselves here by defining a
__parameter__ that lists multiple values of tasks and runs a separate
job step for each value. We do this by adding a `global.parameters`
section at the bottom of the script. We then define individual
parameters within this section. Each parameter includes a list of
`values` (Each element is used in its own job step.) and a `label`.
(The `label` helps define how the output directory structure is
named.)

This is what it looks like to define a global parameter:

```yml
global.parameters:
    TASKS:
        values: [2, 4, 8, 18, 24, 36]
        label: TASKS.%%
```

Note that the label should include `%%` as above; the `%%` is itself a
placeholder!  The directory created for the output of each job step
will be identified by the value of each parameter it used, and the
parameter's value will be inserted to replace the `%%`.

Next, we should update the line under `run` -> `cmd` defining `procs`
to include the name of the parameter enclosed in `$()`:

```yml
          procs: $(TASKS)
```

If we make this change for steps `amdahl-1` _and_ `amdahl-2`, they
will now look _exactly_ the same, so we can simply condense them to
one step.

The full YAML file will look like

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
      P: .999
      OUTPUT: amdahl.json
      OUTPUT_PATH: ./Episode4

study:
    - name: amdahl
      description: run in parallel
      run:
          # Here's where we include our MPI wrapper:
          cmd: |
               $(LAUNCHER) amdahl --terse -p $(P) >> $(OUTPUT)
          nodes: 1
          procs: $(TASKS)
          walltime: "00:00:30"

global.parameters:
    TASKS:
        values: [2, 4, 8, 18, 24, 36]
        label: TASKS.%%
```

::: challenge

Run `maestro run --dry amdahl.yaml` using the above YAML file and
investigate the resulting directory structure. How does the list of
task values under `global.parameters` change the output directory
organization?

::::::solution

Under your current working directory, you should see a directory
structure created with the following format --
`Episode4/Amdahl_<Date>-<Time>/amdahl`.  Within the `amdahl`
subdirectory, you should see one output directory for each of the
values listed for `TASKS` under `global.parameters`:

```bash
ls Episode4/Amdahl_<Date>_<Time>/amdahl
```

```output
TASKS.18  TASKS.2  TASKS.24  TASKS.36  TASKS.4  TASKS.8
```

Each `TASKS...` subdirectory will contain the slurm submission script
to be used if this maestro job is run:

```bash
ls Episode4/Amdahl_<Date>_<Time>/amdahl/TASKS.18/
```

```output
amdahl_TASKS.18.slurm.sh
```

::::::
:::

Run

```bash
maestro run amdahl.yaml
```

before moving on to the next episode, to generate the results for
various task numbers. You'll be able to see your jobs queuing and
running via `squeue -u <username>`.

:::keypoints

- Environment variables are placeholders defined under the `env`
  section of a Maestro YAML.
- Parameters defined under `global.parameters` require lists of
  values and labels.

:::
