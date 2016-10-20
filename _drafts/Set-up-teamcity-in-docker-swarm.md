---
title: Set up TeamCity in a Docker Swarm
author: nieldewet
---

JetBrains have published [TeamCity](https://www.jetbrains.com/teamcity/) images on [Docker Hub](https://hub.docker.com/u/jetbrains/), and with [swarm mode](https://docs.docker.com/engine/swarm/) becoming part of Docker Engine since v1.12.0, why not let the Docker swarm take care of managing your TeamCity server and agents? In this post I will show how to set up TeamCity with with swarm mode on more than one node running in Azure.<!--more-->
