---
title: "Running Maestro on the cluster"
teaching: 30
exercises: 20
---

::: objectives

- "Define rules to run locally and on the cluster"

:::

# How do I run Maestro on the cluster?

What happens when we want to run on the cluster ("to run a batch job")rather
than the login node? The cluster we are using uses Slurm, and Maestro has 
built in support for Slurm. We just need to tell Maestro which resources we
need Slurm to grab for our run.

First, we need to add a `batch` block to our YAML file, where we'll provide the
names of the machine, bank, and queue in which your jobs should run.

```yml
batch:
    type: slurm
    host: pascal # enter the machine you'll run on
    bank: lc # enter the bank to charge
    queue: pvis # enter the partition in which your job should run
```

Second, we need to specify the number of nodes, number of processes, and walltime
for *each study* to be run from our YAML file. This information goes under each
study's `run` field:

```yml
(...)
      run:
          cmd: |
               hostname >> hostname.txt
          nodes: 1
          procs: 1
          walltime: "00:00:30"
```

Here we specify 1 node, 1 process, and a time limit of 30 seconds. **Note** that
the format of the walltime includes quotation marks -- "<Hours>:<Minutes>:<Seconds>".

With these changes, our updated YAML file might look like

```yml
description:
    name: Hostnames
    description: Report a node's hostname.

batch:
    type: slurm
    host: pascal # machine to run on
    bank: lc # bank
    queue: pvis # partition

study:
    - name: hostname-login
      description: Write the login node's hostname to a file
      run:
          cmd: |
              hostname > hostname_login.txt
    - name: hostname_batch
      description: Write the node's hostname to a file
      run:
          cmd: |
               hostname >> hostname.txt
          nodes: 1
          procs: 1
          walltime: "00:00:30"
```

Note that we left the rule `hostname-login` as is. Because we do not specify any info for slurm under this original step's `run` field -- like nodes, processes, or walltime -- this step will continue running on the login node and only `hostname_batch` will be handed off to slurm.

::: challenge
## Running on the cluster

Modify your YAML file, `hostname.yaml` to execute `hostname` on the _cluster_.
Run with 1 node and 1 process using the bank `guest` on the partition
`psummer` on `quartz`.

If you run this multiple times, do you always run on the same node?
(Is the hostname printed always the same?)

:::::: solution

The contents of `hostname.yaml` should look something like:

```yml
description:
    name: Hostnames
    description: Report a node's hostname.

batch:
    type: slurm
    host: quartz # machine to run on
    bank: guest # bank
    queue: psummer # partition

study:
    - name: hostname-login
      description: Write the login node's hostname to a file
      run:
          cmd: |
              hostname > hostname_login.txt
    - name: hostname_batch
      description: Write the node's hostname to a file
      run:
          cmd: |
               hostname >> hostname.txt
          nodes: 1
          procs: 1
          walltime: "00:00:30"

```

When you run this job, a directory called `Hostname_...` will be created. If you look in the subdirectory `hostname_batch`, you'll find a file called `hostname.txt` with info about the compute node where the `hostname` command ran. If you run the job multiple times, you will probably land on different nodes; this means you'll see different node numbers in different hostname.txt files. If you see the same number more than once, don't worry! If you get an answer other than `pascal83`, you're doing it correctly. :)

::::::

:::

## Outputs from a batch job

When running in batch, `maestro run...` will create a new directory with the
same naming scheme as seen in episode 1, and that directory will contain
subdirectories for all studies. The `hostname_batch` subdirectory has four
output files, but this time the file ending with extension `.sh` is a slurm
submission script

```bash
(maestro_venv) janeh@pascal83:~/Hostnames_20240320-170150/hostname_batch$ ls
hostname.err  hostname.out  hostname.slurm.sh  hostname.txt
(maestro_venv) janeh@pascal83:~/Hostnames_20240320-170150/hostname_batch$ cat hostname.slurm.sh
#!/bin/bash
#SBATCH --nodes=1
#SBATCH --partition=pvis
#SBATCH --account=lc
#SBATCH --time=00:00:30
#SBATCH --job-name="hostname"
#SBATCH --output="hostname.out"
#SBATCH --error="hostname.err"
#SBATCH --comment "Write the node's hostname to a file"

hostname > hostname.txt
```

Maestro uses the info from your YAML file to write this script and then
submits it to the scheduler for you. Soon after you run on the cluster via
`maestro run hostname.yaml`, you should be able to see the job
running or finishing up in the queue with the command `squeue -u <your username`.
For example,

```bash
(maestro_venv) janeh@pascal83:~$ maestro run batch-hostname.yaml
[2024-03-20 17:31:37: INFO] INFO Logging Level -- Enabled
[2024-03-20 17:31:37: WARNING] WARNING Logging Level -- Enabled
[2024-03-20 17:31:37: CRITICAL] CRITICAL Logging Level -- Enabled
[2024-03-20 17:31:37: INFO] Loading specification -- path = batch-hostname.yaml
[2024-03-20 17:31:37: INFO] Directory does not exist. Creating directories to /g/g0/janeh/Hostnames_20240320-173137/logs
[2024-03-20 17:31:37: INFO] Adding step 'hostname-login' to study 'Hostnames'...
[2024-03-20 17:31:37: INFO] Adding step 'hostname_batch' to study 'Hostnames'...
[2024-03-20 17:31:37: INFO]
------------------------------------------
Submission attempts =       1
Submission restart limit =  1
Submission throttle limit = 0
Use temporary directory =   False
Hash workspaces =           False
Dry run enabled =           False
Output path =               /g/g0/janeh/Hostnames_20240320-173137
------------------------------------------
Would you like to launch the study? [yn] y
Study launched successfully.
(maestro_venv) janeh@pascal83:~$ squeue -u janeh
             JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
            718308      pvis hostname    janeh  R       0:01      1 pascal13
```

::: keypoints

- "You can run on the cluster by including info for Slurm in your Maestro YAML file"
- "Maestro generates and submits its own batch scripts to your scheduler."
- "Steps without Slurm parameters will run on the login node by default."

:::
