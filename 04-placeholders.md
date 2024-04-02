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

::: callout
## D.R.Y. (Don't Repeat Yourself)

In many programming languages, the bulk of the language features are
there to allow the programmer to describe long-winded computational
routines as short, expressive, beautiful code.  Features in Python,
R, or Java, such as user-defined variables and functions are useful in
part because they mean we don't have to write out (or think about)
all of the details over and over again.  This good habit of writing
things out only once is known as the "Don't Repeat Yourself"
principle or D.R.Y.
:::

Maestro YAML files are a form of code and, in any code, repetition can
lead to problems (e.g. we rename a data file in one part of the YAML
but forget to rename it elsewhere).

In this episode, we'll set ourselves up with ways to avoid repeating
ourselves by using *environment variables* as *placeholders*.


## Placeholders

At the end of our last episode, our YAML file contained the sections

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
          procs: 6
          walltime: "00:01:30"
```

Here we were already using a placeholder -- `$(LAUNCHER)` -- which held
the place and was later swapped out for a call to `srun` specifying
the nodes and tasks wanted.

Let's create another environment variable in the `variables` second under
`env`. We can define a new parallel proportion as `P: .999`. Then, under
`run`'s `cmd`, we can call this environment variable with the syntax
`$(P)`. `$(P)` holds the place of and will be substituted by `.999` when
Maestro creates a Slurm submission script for us

```yml
(...)

env:
    variables:
      P: .999
      OUTPUT_PATH: ./Episode4

study:
    - name: amdahl
      description: run in parallel
      run:
          # Here's where we include our MPI wrapper:
          cmd: |
               $(LAUNCHER) amdahl --terse -p $(P) >> amdahl.json
          nodes: 1
          procs: 6
          walltime: "00:01:30"
```

(Note that the `OUTPUT_PATH` was also updated to reflect the current
episode.)

It may also be helpful to create a variable for our output file, like this:

```yml
(...)

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
          procs: 6
          walltime: "00:01:30"
```

## Maestro's global.parameters

We're almost ready to perform our scaling study -- to see how the amount of work per processor
changes as we use more processors in the job. One way to do this would be to update the line

```yml
          procs: 6
```

and to manually re-run `maestro run...` several times with different numbers of processes.

An alternative is to avoid repeating ourselves by defining a **parameter** that lists multiple
values of tasks and runs a separate job for each value. We do this by adding a 
`global.parameters` section at the bottom of the script. We then list individual parameters
within this section. Each parameter must include a list of its values and a label, using the
following syntax:


```yml
global.parameters:
    TASKS:
        values: [2, 4, 8, 18, 24, 36]
        label: TASKS.%%
```

Note that the parameter is `TASKS` and that its label starts with the same name, but is followed
by a `.%%`. 

We would then update the line under `run` -> `cmd` defining `procs` to include the name
of the parameter enclosed in `$()`:

```yml
          procs: $(TASKS)
```

The full YAML file will look like

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
          walltime: "00:01:30"

global.parameters:
    TASKS:
        values: [2, 4, 8, 18, 24, 36]
        label: TASKS.%%
```

::: challenge

Run `maestro run --dry amdahl.yaml` using the above YAML file
and investigate the resulting directory structure. How does
the list of task values under `global.parameters` change the
output directory organization?

::::::solution

Under your current working directory, you should see a directory structure
created with the following format -- `Episode4/Amdahl_<Date>-<Time>/amdahl`.
Within the `amdahl` subdirectory, you should see one output directory
for each of the values listed for `TASKS` under `global.parameters`:

```bash
(...)Episode4/Amdahl_<Date>_<Time>/amdahl$ ls
```
```output
TASKS.18  TASKS.2  TASKS.24  TASKS.36  TASKS.4  TASKS.8
```

Each `TASKS...` subdirectory will contain the slurm submission script
to be used if this maestro job is run: 

```bash
(...)Episode4/Amdahl_<Date>_<Time>/amdahl$ ls TASKS.18/
```
```output
amdahl_TASKS.18.slurm.sh
```
::::::
:::

Run

```
maestro run amdahl.yaml
```

before moving on to the next episode, to generate the results for various task numbers. You'll be able to
see your jobs queueing and running via `squeue -u <username>`.


:::keypoints
- "Environment variables are placeholders defined under the `env` section of a Maestro YAML."
- "Parameters defined under `global.parameters` require lists of values and labels."
- "Parameters "

:::
