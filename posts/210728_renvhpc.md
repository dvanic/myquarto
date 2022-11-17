---
title: "Installing packages on a PBS-Pro HPC cluster using renv"
author: Darya Vanichkina
date: '2021-07-28'
layout: post
slug: 2021-07-28-renvhpc
categories:
  - Data science
  - R
  - WorkJournal
  - compute
---

## Using renv on a PBS-Pro HPC cluster 

Today someone asked about my workflow for using R on an HPC cluster, and - more specifically - about how to install packages when executing a workflow that requires use of HPC. Google wasn't helpful with providing a link to share - so the onus is on me to write *the* blog post.

![suitcase](https://source.unsplash.com/E2qx9Ed2qIQ/800x600)

### The "why"

Just like working on our local machines, when working on an HPC cluster we really want to be able to have a consistent R environment for each of our projects that we snapshot at the time of working on said project. This means code that was written months or even years ago will still work as expected, even if the packages on CRAN have since been updated with breaking changes. The [`renv`](https://rstudio.github.io/renv/index.html) package is brilliant at managing this - however, it is not intuitive to figure out how to use it when working on an HPC cluster - which is what I work through in the below post.

### Assumptions

This post makes a few assumptions:

1. You are planning to use R for analysis, and know how to install packages on your local machine.
2. You are working with a PBS-Pro HPC cluster, and know all of the core commands (this is not an introduction to `ssh`, `qsub`, `module load` etc - see [here](https://sydney-informatics-hub.github.io/training.artemis.introhpc/) for an intro). I *think* some of the below should generalise to SLURM or other cluster schedulers, but don't have access to them so can't test.
3. You want to use the [`renv`](https://rstudio.github.io/renv/articles/renv.html) library to manage the packages in your project (You do, you really do - I cannot recommend this highly enough, especially for sanity of future you). You know the basics of using `renv` on your local machine, as per [the `renv` vignette](https://rstudio.github.io/renv/articles/renv.html) - that's probably the best intro to `renv`.

Oof, now that that's out of the way, let's dive in :dolphin: ...

### Step 1: `ssh` into your cluster and fire up an interactive session

After you login to your HPC server, I recommend starting up an interactive session so you're not working on and hence clogging up the login nodes. (You can do this on a login node too, but depending on how much compute prototyping your analysis requires this may crash :fire: - or get you a cranky email from an admin :rage: ).

```sh
qsub -I -l walltime=4:0:0 -l select=1:ncpus=1:mem=10gb
```

### Step 2: Launch R and install renv  

After you're logged in to the worker node, run the following commands to:

- navigate to the project directory
- get rid of everything that may have inadvertently been preloaded
- launch R
- install `renv` 

```sh
cd myprojectdirectory
module purge
module load R/4.0.4 # <- or whatver version or R is available on your cluster
R
```

This will open the R command line interface.

```R
install.packages("renv")
```

This will return the details that the installation of `renv()` will occur into a default location:

```sh
Installing package into ‘/home/myusername/R/x86_64-pc-linux-gnu-library/4.0’
```

And ask me to select a CRAN mirror for installation; after choosing the appropriate mirror and pressing return the package will be installed.

### Step 3: Install the packages you need, (ideally) prototype your analysis and snapshot the installs using `renv`

For this demo, I am going to assume that my project depends on the `tidyverse` family of libraries, and that I want to install them in one go. (*In real life, I actually avoid taking this approach, and instead install each of the tidyverse libraries one by one as I use and need them.*)

```R
library(renv)
# initialise renv
renv::init()
```

This will ask me to restart R, after doing which I can start installing packages.

```R
install.packages("tidyverse")
# wait... and wait... for packages to successfully install
library(tidyverse)
```

I would next prototype my analysis code, installing as many packages as I need using the normal `install.packages()` syntax - and eventually wrap the analysis up into a script. 

After you have installed all of the libraries you need, in the interactive R session, exectute the following command:

```R
# this will tell you whether renv has captured versions 
# of all of the packages you need to install
renv::status()
# if the result of renv::status() was not 
# "The project is already synchronized with the lockfile."
# run the below command to snapshot all currently installed libraries
renv::snapshot()
```

For this demo, I'm going to assume I want to run the following command that uses the built-in `gss_cat` dataset from the `forcats` package, the pipe and the `dplyr::filter()` command - so nicely tests a few of the `tidyverse` libraries in one line:

```R
gss_cat %>% head() %>% filter(tvhours < 10)
```

The output should be:

```txt
# A tibble: 3 × 9
   year marital      age race  rincome    partyid     relig     denom    tvhours
  <int> <fct>      <int> <fct> <fct>      <fct>       <fct>     <fct>      <int>
1  2000 Widowed       67 White Not appli… Independent Protesta… No deno…       2
2  2000 Never mar…    39 White Not appli… Ind,near r… Orthodox… Not app…       4
3  2000 Divorced      25 White Not appli… Not str de… None      Not app…       1
```



### (Optional) Step 3a: Look at the `renv.lock` file to see all of the packages you have installed into the project

The `renv.lock` file stores details about the R version and all of the packages that are installed in your environment in `.json` format. So installing the `tidyverse` packages in one fell swoop like I did above results in quite a long list of libraries (last 5 shown here):

```json
 "vroom": {
      "Package": "vroom",
      "Version": "1.5.3",
      "Source": "Repository",
      "Repository": "CRAN",
      "Hash": "aac6012f34348b3ca6bf373fe7172b06"
    },
    "withr": {
      "Package": "withr",
      "Version": "2.4.2",
      "Source": "Repository",
      "Repository": "CRAN",
      "Hash": "ad03909b44677f930fa156d47d7a3aeb"
    },
    "xfun": {
      "Package": "xfun",
      "Version": "0.24",
      "Source": "Repository",
      "Repository": "CRAN",
      "Hash": "88cdb9779a657ad80ad942245fffba31"
    },
    "xml2": {
      "Package": "xml2",
      "Version": "1.3.2",
      "Source": "Repository",
      "Repository": "CRAN",
      "Hash": "d4d71a75dd3ea9eb5fa28cc21f9585e2"
    },
    "yaml": {
      "Package": "yaml",
      "Version": "2.2.1",
      "Source": "Repository",
      "Repository": "CRAN",
      "Hash": "2826c5d9efb0a88f657c7a679c7106db"
    }
```

I sometimes find that `renv::status()` and `renv::snapshot()` in step 3 don't immediately see all of the libraries I have installed in the environment, and - weirdly - I need to restart R and run `renv::status()` again before it will detect that a bunch of libraries have been installed. So if you're missing libraries in the `renv.lock` file you know need to be there - restarting R and re-running `renv::status()` and `renv::snapshot()` may be the way to go.

### Step 4: Save your script and submit it via PBS

Wrap the above code into `myscript.R` , with the following contents:

```R
library(tidyverse)
gss_cat %>% head() %>% filter(tvhours < 10)
```

 Then create a shell script (`myshellscript.sh`) that can be submitted to the PBS queue:

```sh
#! /bin/bash
module purge
module load R/4.0.4
cd /to/my/projectpath
Rscript myscript.R
```

Finally, submit to the queue: 

```bash
qsub -l walltime=0:10:0 -l select=1:ncpus=1:mem=1gb -N mytestscript myshellscript.sh
```

Note that I'm only asking for 1 Gb of RAM and 10 minutes of walltime. This is probably a lot less than you'd need for any "real" analysis - but it's perfectly fine for my script above.

After that executes, I get the following output in `mytestscript.o123456`, where 123456 is the job number that my PBS scheduler automatically assigned to my script run:

```R
# A tibble: 3 × 9
   year marital      age race  rincome    partyid     relig     denom    tvhours
  <int> <fct>      <int> <fct> <fct>      <fct>       <fct>     <fct>      <int>
1  2000 Widowed       67 White Not appli… Independent Protesta… No deno…       2
2  2000 Never mar…    39 White Not appli… Ind,near r… Orthodox… Not app…       4
3  2000 Divorced      25 White Not appli… Not str de… None      Not app…       1
```

And the package loading messages in `mytestscript.e123456`:

```R
Warning message:
renv took longer than expected (66 seconds) to activate the sandbox.

The sandbox can be disabled by setting:

    RENV_CONFIG_SANDBOX_ENABLED = FALSE

within an appropriate start-up .Renviron file. See `?renv::config` for more details.
Warning: program compiled against libxml 209 using older 207
── Attaching packages ─────────────────────────────────────── tidyverse 1.3.1 ──
✔ ggplot2 3.3.5     ✔ purrr   0.3.4
✔ tibble  3.1.3     ✔ dplyr   1.0.7
✔ tidyr   1.1.3     ✔ stringr 1.4.0
✔ readr   2.0.0     ✔ forcats 0.5.1
── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
✖ dplyr::filter() masks stats::filter()
✖ dplyr::lag()    masks stats::lag()
```

This includes a warning that the `renv` sandbox took a while to load, but this doesn't seem to affect functionality, so I haven't disabled it.

***

That's it! Hopefully the above helps you use `renv()` with a PBS-Pro based HPC cluster :blush:, and please do leave a comment if something isn't clear or doesn't work as expected.

### (Even more) notes

1. This workflow means that your package install information (i.e. the binaries) are stored in the project folder, in the `renv` folder. So they are *not* stored on the worker nodes where your job is being executed, but instead in your home or project folder. This is a *good* thing, as it means you're not hitting up CRAN to install packages every time you run a job.
2. This workflow also means that while you can `rsync` your project folder from your local machine to the remove server (or prototype on your local machine), you will need to run `renv::restrore()` once on the "other" machine prior to being able to execute or deploy the script.