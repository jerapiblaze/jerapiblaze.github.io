---
title: Conda Cheetsheet
date: 2023-10-26 17:15:00 +0700
categories: [Blogging, Tutorial]
tags: [anaconda, conda, python]
author: j12t
toc: true
comments: false
math: true
mermaid: false
img_path: /assets/img/posts/2023-10-26-conda-cheatsheet
pin: false
---

## Introduction

With Conda, we can have different setups for different projects. Each setup in Conda is called an environment. In each environment is a seperated space with its own packages/executables/libraries.

In this tutorial, we will have a quick guide to commonly used command to manage and use Conda environment. If you are looking for real-world example, just jump to [this section](#real-world-examples).


## Using environments

Here are some commonly used action within an Conda environment:

- Select and use an environment: `conda activate environment_name`.

- Install packages: `conda install package1_name package2_name -c conda-forge`.
<br>Most of popular Python packages can be installed this way.
<br>If Conda says it cannot find the packages, we need to find the exact command to install it, by visit the [Anaconda Package Site](https://anaconda.org/anaconda/repo), search for the package and copy the command.

- Remove packages: `conda uninstall package1_name package2_name`

Python is a package in Conda, so it can be upgraded/downgraded easily. The only thing we need to do is tell Conda which version of Python to use in the environment: `conda install python=x.xx`.

## Managing environments

Here are some commonly used command to manage Conda environments:

- Create an new environment: `conda create -n environment_name`.

- Create new environment with initial packages: `conda create -n environment_name package1_name package2_name -c conda-forge`.

- Get all of created environment on the current machine: `conda env list`.

- To delete an environment: `conda remove -n environment_name`.

It is recommended that we create seperated environment for each projects, and **never modify the `base` environment**.

## Real-world examples

In this section, we will demonstates the procedure to setup and use Conda with Visual Studio Code to work with Python and Python Jupyter Notebook.

At the very first day of the project, we created an new setup.

```bash
# Create a new folder for the project and jump in to it 
mkdir example_project
cd example_project
# Setup the environment
conda create -n example_project python=3.10 ipykernel -y
# Install  `networkx`, `mathplotlib` and `numpy`
conda install networkx mathplotlib numpy -c conda-forge -y
# Open Visual Studio Code
code .
# Happy coding.
```

On a sunny day with no clouds, we are suddenly informed that we need to use Python 3.12 to synchonize with the rest of the big team, we just need to tell Conda to do that for us by:
```bash
conda install python=3.12 -y
```

When the project finished the formulation stage, we found that the package `numpy` is no longer needed, so we uninstall it:
```bash
conda uninstall numpy -y
```

After that, we found that we need to use `cplex`, but when using regular command Conda said that it unable to find the package. So we take a little more time to Google search `cplex conda package`, and we have the right command to install:
```bash
conda install cplex -c ibmdecisionoptimizations
```

Another day, our project manager decided to revert to Python 3.10 for maximum compathbility. Other group panicked because they have to uninstall Python 3.12, and install Python 3.10 and then reinstall all of packages. During that time, we just sit in a coffee room and run a single commandand after a few minutes, we can back to work:
```bash
conda install python=3.10 -y
```

## Summarize

You reached to the end of the post! Now we hope you now already have a basic knowledge about common commands to manage and use Conda environments. Happy scienctific coding.
