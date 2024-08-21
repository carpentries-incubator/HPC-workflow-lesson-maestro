# HPC Workflow Management with Maestro

## The lesson

"HPC Workflow Management with Maestro" was written to follow HPC Carpentry's 
[HPC Intro](https://carpentries-incubator.github.io/hpc-intro/), where learners get hands-on
experience working with Slurm and running a parallel program called
[`amdahl`](https://github.com/hpc-carpentry/amdahl). In this lesson, learners will see how
workflow management tools (here, `maestro`) can automate the user's job submission processes,
leaving behind a record of how and when the user's work was completed. This automation and
record making helps users to document and reproduce their workflows.

This lesson was originally ported to `maestro` from HPC Carpentry's 
[workflow lesson with snakemake](https://carpentries-incubator.github.io/hpc-workflows/) for a
[workshop](https://www.hpc-carpentry.org/blog/2024/08/llnl-workshop-blog-post.html) at
Lawrence Livermore National Lab.

## About Maestro

From Maestro's [documentation](https://maestrowf.readthedocs.io/en/latest/) and [GitHub](https://github.com/LLNL/maestrowf):

> Maestro  is an open-source HPC software tool that defines a YAML-based study specification for
> defining multistep workflows and automates execution of software flows on HPC resources. The
> core design tenants of Maestro focus on encouraging clear workflow communication and documentation,
> while making consistent execution easier to allow users to focus on science. 
>
> Maestro gives an easy path to automating and orchestrating your workflows, building upon
> your existing shell and batch (HPC scheduled scripts/tasks) script tasks to layer on
> parameterization, task dependencies, and output isolation. Additionally, Maestro's workflow
> specification layer enables documenting those scripts and their interdependencies if chaining
> them together, and makes them more repeatable and shareable for enhanced collaboration with
> your peers.

You can learn more about other workflow tools [here](https://workflows.community/systems).

## Building these pages locally

These lesson pages are built using the Carpentries Workbench, powered by the R package and
static website generator [sandpaper](https://carpentries.github.io/sandpaper-docs/). You can
install sandpaper and render these pages locally with the following instructions.

1. Install R & RStudio [here](https://posit.co/download/rstudio-desktop/#download).
Rstudio includes `pandoc` another requirement for building these docs.

2. In RStudio's console, run

```
# register the repositories for The Carpentries and CRAN
options(repos = c(
  carpentries = "https://carpentries.r-universe.dev/",
  CRAN = "https://cran.rstudio.com/"
))

# Install the template packages to your R library
install.packages(c("sandpaper", "varnish", "pegboard", "tinkr"))
```

3. From RStudio, use the menu to change your working directory
to your local/cloned copy of this repo via 
**Session > Set Working Directory > Choose Directory**.

4. In RStudio's console, run

```
library("sandpaper")
sandpaper::serve()
```

5. Copy `http://127.0.0.1:4321` from the output printed to
RStudio's console and paste it into the browser. Your rendered
docs should be visible from there!


For more info, see https://carpentries.github.io/sandpaper-docs/

## Contributing

Contributions to this lesson are welcome! Before submitting a PR, please build the web pages
locally and check that they build correctly with your changes. 
