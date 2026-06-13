---
title: Chào thế giới LaTeX
date: 2024-12-11 10:00:00 +0700
categories: [Blogging, Others]
tags: [helloworld]
author: j12t
toc: true
comments: false
math: true
mermaid: false
# image:
#     path: download.png
#     alt: random image
pin: false
---

## How to download NASA-MERRA2 folder

First, create an account on [EarthData](https://urs.earthdata.nasa.gov/)

Remeber your `username` and `password`.

Then, download the file list: [`mrr2_1980_2023.txt`](https://j12tee.qzz.io). You can use your own list from EarthData.

Install `aria2` downloader:
```bash
sudo apt install aria2
```
If you do not have root access, consider using pre-built binaries or compile it from source. Check [`aria2` homepage](https://aria2.github.io/) for more details.

Modify the following script, save it as `download_merra2.sh`:

```bash
#!/bin/bash
DEST=/path/to/download/folder
LOG=/path/to/a/file
COOKIES=/path/to/a/file
USER=replace_with_your_earthdata_username
PASS=replace_with_your_earthdata_password
LIST=/path/to/mrr2_1980_2023.txt/file
aria2c --http-user="$USER" --http-passwd="$PASS" --save-cookies="$COOKIES" --load-cookies="$COOKIES" --log="$LOG" --log-level="notice" --content-disposition --continue=true --check-integrity=true --dir="$DEST" --input-file="$LIST"
```

Make the script executable and run it:

```bash
chmod +x download_merra2.sh
./download_merra2.sh
```

The download progress should run immediatelty. Due to large data volume (about 20TB), it can take very long to finish. If the download progress is interupted, you can resume the download progress by re-executing the script.
