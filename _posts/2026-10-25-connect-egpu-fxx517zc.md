---
title: Reliable way to connect e-GPU to Asus TUF FX517 via Thunderbolt 4
date: 2026-10-25 11:11:11 +0100
categories: [Blogging, Tutorial]
tags: [windows, egpu]
author: j12t
toc: true
comments: true
math: true
mermaid: true
media_subpath: /assets/img/posts/2026-10-25-connect-egpu-fxx517zc
# image:
#     path: download.png
#     alt: random image
pin: false
---

## Why?

I just bought a new eGPU, just for curiosity. After many reboots, EC reset, driver reinstalls, ... I finally found a reliable way to connect and disconnect the eGPU on my [TUF Dash F15](https://www.notebookcheck.net/Asus-TUF-Dash-F15-Series.520628.0.html){:target="_blank"} using the [Thunderbolt 4](https://en.wikipedia.org/wiki/Thunderbolt_(interface)#Thunderbolt_4){:target="_blank"} port. I think other models with same TB4 port configuration will share the same procedure.

> Tl;dr: Never let 2 NVIDIA GPUs appear at the same time.

## How

> You will need to connect your power adapter, and it should be connected throughout the process. The Type-C PD cannot give enough power to the hungry CPU.
{: .prompt-info }

### Preparation

First of all, use [G-Helper](https://github.com/seerge/g-helper){:target="_blank"} instead of Armoury Crate. Uninstall AC completely using [AC uninstall tool](https://dlcdnets.asus.com/pub/ASUS/mb/14Utilities/Armoury_Crate_Uninstall_Tool.zip){:target="_blank"} and switch to G-Helper.

Using G-Helper, disable all Asus services (very important, since these nasty things will prevent the initailization of the eGPU).

> Install the latest original driver from ASUS homepage. Do not use any drivers which is newer than the drivers from ASUS page. Ignore driver updates from NVIDA app!
{: .prompt-warning }

Now, you are ready to connect.

### Connect

To connect the eGPU, do the followings:

- Switch GPU mode to `Eco`. Wait until the dGPU is completely disappear from `Device Manager`.
  ![Switch to `Eco` to disable the dGPU](ghelper-eco.png)
- Connect the eGPU using the one and only Thunderbolt 4 port, which is the USB-C port near the HDMI port.
- Wait until the eGPU is fully functional. This process can take some minutes, and the eGPU will have `code 18` for a while.
- DO NOT disconnect the eGPU disruptly.

After finishing these steps, your eGPU will be ready to use. However, the remaining USB-C port and the HDMI port will not able to connect to external displays, since these ports connect to the dGPU which is now disabled.

![Happy eGPU](egpu-devicemgmt.png)

### Disconnect

To disconnect the eGPU, do the followings:

- Disable the eGPU in `Device Manager`. Wait until it is completely stop working (fans stop moving).
  ![Disable eGPU in Device Manager](egpu-disable.png)
- Disconnect the TB4 cable.
- Switch GPU mode to `Standard`.
- Wait for a while until the dGPU is fully functional. This process can take some minutes, and the dGPU may have `code 18`. If it take too long, reinstall the driver and reboot. Most of the case, you do not need to do so and the dGPU will be available after a short time.

![dGPU has `code 18`](dgpu-code18.png)

Great, now your dGPU, HDMI port, lower USB-C port are happy to work again.

> You may need to restart GPU-Z and other hardware monitor tools in order to make they work properly again.
{: .prompt-info }

![Happy dGPU](dgpu-devicemgmt.png)

### Some notes

Note about **power strip off**: Since the laptop now only have to handle power and cooling of only CPU, the CPU can become very power hungry, consume lots of power and give your board's power cricuits a headache. We all know the FX517 series has very limited cooling capability, so do not be suprise if your laptop trigger hard power off, it can be due to overheat somewhere in the board. Just for precaution, use the power limit function of G-Helper.

![Power limit in G-Helper](ghelper-powerlimit.png)

Note about **PCIe speeds**: Although via GPU-Z, you will see your GPU is connected with PCIe 4 x4 or even 5 x4, it is actually connect through TB4 PCIe tunnel, which is capped at 40Gbps down/40Gbps up (equivalent to PCIe 3x4), see screenshots below. So the performance gain will be... very vavies compared to the dGPU. If you are hoping a better gain in FPS gaming, no it will not help due to high latency. If you do not need any low-latency and just want a higher frame rate and better image quality, the eGPU will be a costly but worth solution. Or even better, local SLMs/LLMs, but in this RAMpocalype era, you will probaly need to pay a great price.

![GPU-Z check](gpuz.png)

![Actual speed, reported in Settings](actualspeeds.png)

## End

Happy tinkering. Hope this post help you in some way.
