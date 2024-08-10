---
title: Setup Anaconda and Visual Studio Code to begin with scientific Python
date: 2023-10-26 17:15:00 +0700
categories: [Blogging, Tutorial]
tags: [anaconda, conda, vscode, python]
author: j12t
toc: true
comments: true
math: true
mermaid: false
img_path: /assets/img/posts/2023-10-26-setup-anaconda-vscode-python
image:
    path: download.png
    alt: random image
pin: false
---

# Introduction

**WHAT** is Conda? Conda is a distribution of Python for scientific programming that aims to simpily package management and deployment.

**WHY** Conda? Conda is easy to use and setup. One of the best alternatives is `virtualenv`.

Conda makes installing packages between projects easily. With Conda, we can have multiple package environment with different Python version for each project. Conda also handles package dependances, support upgrade/downgrade and resolve package conflicts, which will helps users focus on science works rather than try hard just to setup working environment.

In this tutorial, we will have a quick guide to setup Conda to work with Visual Studio Code, one of the most popular code-editors.

# Instructions

## Preparation

First, we need to download [Anaconda](https://www.anaconda.com/download/) and [Virual Studio Code](https://code.visualstudio.com/Download) installers.

Then we need to execute the installers, just use default options.

### Windows

On Windows, we only need to double-click on the `.exe` installers, hit `next` continuously until `finish`.

-----------

### Ubuntu

On Linux, in other hand, is a little bit harder and procedures depends on the distros. In this tutorial, we will demonstrade with `Ubuntu`. 

1. Add execute permission to Anaconda installer: `chmod +x /path/to/anaconda/installer`
2. Execute Anaconda installer `sh /path/to/anaconda/installer`
3. Just follow the instructions
4. Install Visual Studio Code `sudo apt install /path/to/vscode/installer -y`


-----------

At this point, we now have a out-of-box working Anaconda and Visual Studio Code. Then, we need to configure them to work with each other.

## Setup Conda

Just run the initialization command of Conda.

```bash
conda init --all
```

## Setup Vscode with Conda

```bash
code --install-extension mechatroner.rainbow-csv --force
code --install-extension ms-python.python --force
code --install-extension ms-python.vscode-pylance --force
code --install-extension ms-toolsai.jupyter --force
code --install-extension ms-toolsai.jupyter-keymap --force
code --install-extension ms-toolsai.jupyter-renderers --force
code --install-extension ms-toolsai.vscode-jupyter-cell-tags --force
code --install-extension ms-toolsai.vscode-jupyter-slideshow --force
code --install-extension ms-vscode-remote.remote-ssh --force
code --install-extension ms-vscode-remote.remote-ssh-edit --force
code --install-extension ms-vscode.remote-explorer --force
code --install-extension ms-vscode.remote-server --force
code --install-extension msrvida.vscode-sanddance --force
code --install-extension RandomFractalsInc.vscode-data-preview --force
```

# Summarize

Huh, no images? Yeah, as this tutorial is mainly done on terminals, I am so lazy to post images.
