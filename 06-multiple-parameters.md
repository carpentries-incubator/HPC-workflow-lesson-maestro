---
title: "Multiple parameters"
teaching: 50
exercises: 30
---

::: questions
- "How do I specify multiple parameters?"
- "How do multiple parameters interact?"
:::

::: objectives
- "Create scaling results for different proportions of parallelizable code."
- "Create and compare scalability plots for codes with different amounts of parallel work."
:::

## Adding a second parameter

In this episode, we want to vary `P`, the fraction of parallel code,
as part of our workflow. To do this, we will add a second entry under
`global.parameters` and remove the definition for `P` under `env`:

```yml
(...)
env:
    variables:
      OUTPUT: amdahl.json
      OUTPUT_PATH: ./Episode6

(...)

global.parameters:
    TASKS:
        values: [2, 4, 8, 16, 32]
        label: TASKS.%%
    P:
        values: [<Insert values>]
        label: P.%%
```

How many values do we want to include for `P`? We need to have
**the same number** of values listed for all `global.parameters`.
This means that to run the previous scaling study with `P=.5`,
and five values for `TASKS`, our global parameters section would
specify `.5` for `P` five times:

```yml
global.parameters:
    TASKS:
        values: [2, 4, 8, 16, 32]
        label: TASKS.%%
    P:
        values: [.5, .5, .5, .5, .5]
        label: P.%%
```

Let's say we want to perform the same scaling study for a second
value of P, `.99`. This means that we'd have to repeat the same
5 values for `TASKS` and then provide the second value of `P` 5
times:

```yml
global.parameters:
    TASKS:
        values: [2, 4, 8, 16, 32, 2, 4, 8, 16, 32]
        label: TASKS.%%
    P:
        values: [.5, .5, .5, .5, .5, .99, .99, .99, .99, .99]
        label: P.%%
```

At this point, our entire YAML file should look something like

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
      OUTPUT: amdahl.json
      OUTPUT_PATH: ./Episode6

study:
    - name: run-amdahl
      description: run in parallel
      run:
          # Here's where we include our MPI wrapper:
          cmd: |
               $(LAUNCHER) amdahl --terse -p $(P) >> $(OUTPUT)
          nodes: 1
          procs: $(TASKS)
          walltime: "00:01:30"
    - name: plot
      description: Create a plot from `amdahl` results
      run:
          cmd: |
               python3 $(SPECROOT)/plot_terse_amdahl_results.py output.jpg $(run-amdahl.workspace)/TASKS.*/amdahl.json
          depends: [amdahl_*]

global.parameters:
    TASKS:
        values: [2, 4, 8, 16, 32, 2, 4, 8, 16, 32]
        label: TASKS.%%
    P:
        values: [.5, .5, .5, .5, .5, .99, .99, .99, .99, .99]
        label: P.%%
```

::: challenge

Run the workflow above. Do you generate `output.jpg`? If not, why not?

::::::solution

If you use the YAML above, your `amdahl-run` steps should work, but your
`plot` step will fail. `plot`'s failure will be evident both because
`output.jpg` will be missing from the `plot` subdirectory **and**
because the `plot.*.err` file in the same directory will contain an
error:

```
Traceback (most recent call last):
  File "/p/lustre1/janeh/test-maestro/snakemake-port/ep6/plot_terse_amdahl_results.py", line 46, in <module>
    process_files(filenames, output=output)
  File "/p/lustre1/janeh/test-maestro/snakemake-port/ep6/plot_terse_amdahl_results.py", line 10, in process_files
    with open(filename, 'r') as file:
FileNotFoundError: [Errno 2] No such file or directory: '/g/g0/janeh/Episode6/Amdahl_20240328-163359/run-amdahl/TASKS.*/amdahl.json'
```

The problem is that the directory path for `.json` files has changed.
This will be discussed more below!
::::::
:::

The trouble with the YAML above is that our output directory
structure changed when we added a second global parameter, but
we didn't update the directory path specified under `plot`.

If we look inside the `run-amdahl` output folder (identified
as `$(run-amdahl.workspace)` in our workflow YAML), its
subdirectory names now include information about both
global parameters:

```
janeh@pascal83:~/Episode6/Amdahl_20240328-163359/run-amdahl$ ls
P.0.5.TASKS.16  P.0.5.TASKS.8   P.0.99.TASKS.4
P.0.5.TASKS.2   P.0.99.TASKS.16  P.0.99.TASKS.8
P.0.5.TASKS.32  P.0.99.TASKS.2
P.0.5.TASKS.4   P.0.99.TASKS.32
```

whereas directory path for our `.json` files is specified
as `$(run-amdahl.workspace)/TASKS.*/amdahl.json` under the `plot`
step.

We could get the `plot` step to work by simply
adding a wildcard, `*`, in front of `TASKS` so that the
path to `.json` files would be

```
$(run-amdahl.workspace)/*TASKS.*/amdahl.json
```

and the definition for `plot` would be

```
    - name: plot
      description: Create a plot from `amdahl` results
      run:
          cmd: |
               python3 $(SPECROOT)/plot_terse_amdahl_results.py output.jpg $(run-amdahl.workspace)/*TASKS.*/amdahl.json
          depends: [amdahl_*]
```

This would allow `plot` to terminate happily and to produce
an `output.jpg` file, but that image would plot output from
all `.json` files as a single line, and we wouldn't be able
to tell which data points corresponded to a parallel fraction,
`P`, of `.85` and which corresponded to `P=.99`.

If we can generate two plots -- one for each value of `P`
-- we'll more clearly be able to see scaling behavior for
these two situations. We can generate two separate plots
by calling `python3 plot_terse_amdahl_results.py ...` on
two sets of input files -- those in the `P.0.5.TASKS*`
subdirectories of `$(run-amdahl.workspace)` and those
in the `P.0.99.TASKS*` subdirectories.

That means we can generate these two plots by inserting
the variable `$(P)` into the path ---

```
$(run-amdahl.workspace)/P.$(P).TASKS.*/amdahl.json
```

making the definition for `plot`

```
    - name: plot
      description: Create a plot from `amdahl` results
      run:
          cmd: |
               python3 $(SPECROOT)/plot_terse_amdahl_results.py output.jpg $(run-amdahl.workspace)/P.$(P).TASKS.*/amdahl.json
          depends: [amdahl_*]
```

::: challenge

Modify your workflow as discussed above to generate
output plots for two different values of P. Open these
plots and verify they are different from each other.
How does changing the workflow to generate two separate
plots change the directory structure?

(Feel free to use .5 and .99 or to modify to
other values of your choosing.)

:::::: solution

Your full YAML file should be similar to

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
      OUTPUT: amdahl.json
      OUTPUT_PATH: ./Episode6

study:
    - name: run-amdahl
      description: run in parallel
      run:
          # Here's where we include our MPI wrapper:
          cmd: |
               $(LAUNCHER) amdahl --terse -p $(P) >> $(OUTPUT)
          nodes: 1
          procs: $(TASKS)
          walltime: "00:01:30"
    - name: plot
      description: Create a plot from `amdahl` results
      run:
          cmd: |
               python3 $(SPECROOT)/plot_terse_amdahl_results.py output.jpg $(run-amdahl.workspace)/P.$(P).TASKS.*/amdahl.json
          depends: [amdahl_*]

global.parameters:
    TASKS:
        values: [2, 4, 8, 16, 32, 2, 4, 8, 16, 32]
        label: TASKS.%%
    P:
        values: [.5, .5, .5, .5, .5, .99, .99, .99, .99, .99]
        label: P.%%
```

Modifying `plot` to include `$(P)` caused this step to run twice.
As a result, two subdirectories under `plot` were created --
one for each value of P:

```
(maestro_venv) janeh@pascal83:~/Episode6/Amdahl_20240328-171343/plot$ ls
P.0.5  P.0.99
```
::::::
:::

::: callout

Instead of modifying the path to our `amdahl.json` files to
`$(run-amdahl.workspace)/P.$(P).TASKS.*/amdahl.json`, we
could have equivalently updated it to
`$(run-amdahl.workspace)/$(P.label).TASKS.*/amdahl.json`.

In other words, `P.$(P)` is equivalent to `$(P.label)`.
Similarly, in Maestro, `TASKS.$(TASKS)` is equivalent to
`$(TASKS.label)`. This syntax works for every global parameter
in Maestro.

:::

::: keypoints
- "Multiple parameters can be defined under `global.parameters`."
- "Lists of values for all global parameters must have the same
length; the Nth entries in the lists of values for all global
parameters are used in a single job."
:::
