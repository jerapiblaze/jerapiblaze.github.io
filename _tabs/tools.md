---
icon: fa-solid fa-toolbox
title: My tiny-tools
layout: post
order: 2
toc: true
math: true
mermaid: true
---

This is my list of home-made tools, built for my own need.

## Micmute-indicator

I often forgot to press "Unmute" button on calls or meetings due to there're no visual indicator on my laptop. This is the indicator for "Muted" state, only shown when there's an app using the microphone.

![micmute-indicator in action](https://raw.githubusercontent.com/jerapiblaze/micmute-indicator/v1.0/docs/img/demo.png)

<i class="fa-brands fa-github"></i> [Source code](https://github.com/jerapiblaze/micmute-indicator)

## Hust-hotspot-autologin

I'm tired about our lab's internal network at HUST being disconnected and requiring manual login every 24 hours. This script will check if login is needed and automatically send login request without the need of entering username/password. To make it run periodicly, use `cronjob` (for Linux friends) or `Task Scheduler` (for Windows friends).

![hust-hotspot-autologin source code](https://raw.githubusercontent.com/jerapiblaze/hust-hotspot-autologin/main/docs/demobash.png)

<i class="fa-brands fa-github"></i> [Source code](https://github.com/jerapiblaze/hust-hotspot-autologin)
