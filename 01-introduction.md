---
title: "Running commands with Maestro"
teaching: 30
exercises: 30
---

::: questions
- "How do I run a simple command with Maestro?"
:::

:::objectives
- "Create a Maestro YAML file"
:::


## What is the workflow I'm interested in?

In this lesson we will make an experiment that takes an application which runs
in parallel and investigate it's scalability. To do that we will need to gather
data, in this case that means running the application multiple times with
different numbers of CPU cores and recording the execution time. Once we've
done that we need to create a visualisation of the data to see how it compares
against the ideal case.

From the visualisation we can then decide at what scale it
makes most sense to run the application at in production to maximise the use of
our CPU allocation on the system.

We could do all of this manually, but there are useful tools to help us manage
data analysis pipelines like we have in our experiment. Today we'll learn about
one of those: Maestro.

In order to get started with Maestro, let's begin by taking a simple command
and see how we can run that via Maestro. Let's choose the command `hostname`
which prints out the name of the host where the command is executed:

```bash
janeh@pascal83:~$ hostname
```
```output
pascal83
```

That prints out the result but Maestro relies on files to know the status of
your workflow, so let's redirect the output to a file:

```bash
janeh@pascal83:~$ hostname > hostname_login.txt
```

## Writing a Maestro YAML

Edit a new text file named `hostname.yaml`.

Contents of `hostname.yaml`:

```yml
description:
    name: Hostnames
    description: Report a node's hostname.

study:
    - name: hostname-login
      description: Write the login node's hostname to a file
      run:
          cmd: |
              hostname > hostname_login.txt
```

::: callout

## Key points about this file

1. The name of `hostname.yaml` is not very important; it gives us information
   about file contents and type, but maestro will behave the same if you rename
   it to `hostname` or `foo.txt`.
1. The file specifies fields in a hierarchy. For example, `name`, `description`,
   and `run` are all passed to `study` and are at the same level in the hierarchy.
   `description` and `study` are both at the top level in the hierarchy. 
1. Indentation indicates the hierarchy and should be consistent. For example, all
   the fields passed directly to `study` are indented relative to `study` and
   their indentation is all the same. 
1. The commands executed during the study are given under `cmd`. Starting this
   entry with `|` and a newline character allows us to specify multiple commands.
1. The example YAML file above is pretty minimal; all fields shown are required.
1. The names given to `study` can include letters, numbers, and special characters.


:::

Back in the shell we'll run our new rule. At this point, we may see an error if
a required field is missing or if our indentation is inconsistent.

```bash
$ maestro run hostname.yaml
```

::: callout

## `bash: maestro: command not found...`

If your shell tells you that it cannot find the command `maestro` then we need
to make the software available somehow. In our case, this means activating the
python virtual environment where maestro is installed.
```bash
source /usr/global/docs/training/janeh/maestro_venv/bin/activate
```

You can tell this command has already been run when `(maestro_venv)` appears
before your command prompt:


```bash
janeh@pascal83:~$ source /usr/global/docs/training/janeh/maestro_venv/bin/activate
(maestro_venv) janeh@pascal83:~$
```

Now that the `maestro_venv` virtual environment has been activated, the `maestro`
command should be available, but let's double check

```bash
(maestro_venv) janeh@pascal83:~$ which maestro
```
```output
/usr/global/docs/training/janeh/maestro_venv/bin/maestro
```
:::


## Running maestro

Once you have `maestro` available to you, run `maestro run hostname.yaml`
and enter `y` when prompted

```bash
(maestro_venv) janeh@pascal83:~$ maestro run hostname.yaml
[2024-03-20 15:39:34: INFO] INFO Logging Level -- Enabled
[2024-03-20 15:39:34: WARNING] WARNING Logging Level -- Enabled
[2024-03-20 15:39:34: CRITICAL] CRITICAL Logging Level -- Enabled
[2024-03-20 15:39:34: INFO] Loading specification -- path = hostname.yaml
[2024-03-20 15:39:34: INFO] Directory does not exist. Creating directories to /g/g0/janeh/Hostnames_20240320-153934/logs
[2024-03-20 15:39:34: INFO] Adding step 'hostname-login' to study 'Hostnames'...
[2024-03-20 15:39:34: INFO]
------------------------------------------
Submission attempts =       1
Submission restart limit =  1
Submission throttle limit = 0
Use temporary directory =   False
Hash workspaces =           False
Dry run enabled =           False
Output path =               /g/g0/janeh/Hostnames_20240320-153934
------------------------------------------
Would you like to launch the study? [yn] y
Study launched successfully.
```

and look at the outputs. You should have a new directory whose name includes a
date and timestamp and that starts with the `name` given under `description`
at the top of `hostname.yaml`.

In that directory will be a subdirectory for every `study` run from
`hostname.yaml`. The subdirectories for each study include all output files
for that study

```bash
(maestro_venv) janeh@pascal83:~$ cd Hostnames_20240320-153934/
(maestro_venv) janeh@pascal83:~/Hostnames_20240320-153934$ ls
```
```output
batch.info      Hostnames.pkl        Hostnames.txt  logs  status.csv
hostname-login  Hostnames.study.pkl  hostname.yaml  meta
```
```bash
(maestro_venv) janeh@pascal83:~/Hostnames_20240320-153934$ cd hostname-login/
(maestro_venv) janeh@pascal83:~/Hostnames_20240320-153934/hostname-login$ ls
```output
hostname-login.2284862.err  hostname-login.2284862.out  hostname-login.sh  hostname_login.txt
```

::: challenge

To which file will the login node's hostname, `pascal83`, be written?

1. hostname-login.2284862.err
2. hostname-login.2284862.out
3. hostname-login.sh
4. hostname_login.txt

:::::: solution
(4) hostname_login.txt

In the original `hostname.yaml` file that we ran, we specified that
hostname would be written to `hostname_login.txt`, and this is where
we'll see that output, if the run worked!
::::::
:::

::: challenge

This one is tricky! In the example above, `pascal83` was written to
`.../Hostnames_{date}_{time}/hostname-login/hostname_login.txt`.

Where would `Hello` be written for the following YAML?

```yml
description:
    name: MyHello
    description: Report a node's hostname.

study:
    - name: give-salutation
      description: Write the login node's hostname to a file
      run:
          cmd: |
              echo "hello" > greeting.txt
```


1. `.../give-salutation_{date}_{time}/greeting/greeting.txt`
2. `.../greeting_{date}_{time}/give_salutation/greeting.txt`
3. `.../MyHello_{date}_{time}/give-salutation/greeting.txt`
4. `.../MyHello_{date}_{time}/greeting/greeting.txt`

:::::: solution

(3) `.../MyHello_{date}_{time}/give-salutation/greeting.txt`

The toplevel folder created starts with the `name` field under `description`; here, that's `MyHello`.
Its subdirectory is named after the `study`; here, that's `give-salutation`.
The file created is `greeting.txt` and this stores the output of `echo "hello"`.

::::::
:::

::: keypoints

- "You execute `maestro run` with a YAML file including information about your run."
- "Your run includes a description and at least one study (a step in your run)."
- "Your maestro run creates a directory with subdirectories and outputs for each study."

:::
