---
title: Kubernetes App Upgrade Tip
parent: 2021
tags: kubernetes upgrades
---
# Kubernetes App Upgrade Tip

## Major version upgrades

Makesure you disable your health probes when doing a major version upgrade, this is to ensure any schema updates that may delay the startup can sucessfully run without the health checks killing the process and going into a crashloop or even worse cause corruption.

![image](https://user-images.githubusercontent.com/13536174/138124298-fa192d14-fd25-4c01-9c83-07204e968330.png)

Just comment out httpget, path & port and add exec, command ls. I did use to adjust the timers, but I think this way is more eloquent. 

#### 20th of October 2021
