---
title: Setup part-time headless Jetson Orin
date: 2026-10-21 11:11:11 +0100
categories: [Blogging, Tutorial]
tags: [ubuntu, jetson orin, headless]
author: j12t
toc: true
comments: true
math: true
mermaid: true
# image:
#     path: download.png
#     alt: random image
pin: false
---

## Why?

Remote solutions like RustDesk requires a monitor to work. Although there is headless option, it seems to work not steadily. Further, most GPUs will disable their render engines if no physical monitor is attached (to save power, I hate that). I also know about the dummy plugs, and in fact, I brought a bunch of HDMI dummies and used in many of my setups. That's the problem for my Jetson Orin, since it only has DP.

You may say, why not `xrdp`? Well, `xrdp` with `egfx` is only available on `Ubuntu 26.04` or later, and `xrdp-egfx` is not available for `arm64` on `Ubuntu 24.04`. And guess what, `Ubuntu 24.04` is the latest we can put on Jetson Orin, currently.

## How

NVIDIA's X driver can start without a connected monitor by creating a virtual screen. When a monitor is plugged in, the real display is used automatically.

First, create the config file

```bash
sudo mkdir -p /etc/X11/xorg.conf.d
sudo nano /etc/X11/xorg.conf.d/10-nvidia-headless.conf
```

Now insert content of the config file

```apache
Section "Device"
    Identifier "NVIDIA0"
    Driver "nvidia"
    Option "AllowEmptyInitialConfiguration" "true"
EndSection

Section "Screen"
    Identifier "Screen0"
    Device "NVIDIA0"
    DefaultDepth 24

    SubSection "Display"
        Depth 24
        Virtual 1920 1080
    EndSubSection
EndSection
```

Restart the display (if you are logged in in-person, you will be logged out!!!)

```bash
sudo systemctl restart gdm3
```

Or just perform a reboot.

That's it. Enjoy your part-time headless setup.
