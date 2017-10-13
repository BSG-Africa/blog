---
title: Linux - Installing alternative versions of docker client
author: nieldewet
---

Sometimes one needs more than one version of the docker client in order to communicate with different versions of the docker engine, for example that may be running in different production environments.

In this short entry I will show how to use the `update-alternatives` utility to manage multiple versions of the docker client.<!--more-->

#### Setting up alternatives
##### Step 1: Move your existing `docker` binary
Suppose you have [installed](https://docs.docker.com/engine/installation/) the latest version of docker on your machine. First, move the binary to another location. For example:

    sudo mv /usr/bin/docker /usr/bin/docker-117

##### Step 2: Download needed version of docker
Download the needed version of the docker binaries, for example: [https://get.docker.com/builds/Linux/x86_64/docker-1.13.1.tgz](https://get.docker.com/builds/Linux/x86_64/docker-1.13.1.tgz)

You can see all the available releases at [https://github.com/moby/moby/releases](https://github.com/moby/moby/releases)

##### Step 3: Extract the `docker` binary
Extract and move the docker binary to a convenient location, for example:

    tar xvf docker-1.13.1.tgz docker/docker
    mkdir -p ${HOME}/tools/docker-113
    mv docker/docker ${HOME}/tools/docker-113

##### Step 4: Set up alternatives for docker
First, install an alternative for your latest docker client. In our case that is `/usr/bin/docker-117`.

    sudo update-alternatives --install /usr/bin/docker docker /usr/bin/docker-117 10

Using a high priority number will ensure that this alternative is chosen in 'auto' mode.

Next, install an alternative for the other version of the docker client, using the location in Step 3.

    sudo update-alternatives --install /usr/bin/docker docker ${HOME}/tools/docker-113/docker 1

#### Usage
Now you can easily choose which version of docker you wish to use. Simply run this command and interactively select the correct alternative:

    sudo update-alternatives --config docker

Check the version of docker:

    docker -v

To reset your docker client to the highest priority version, run:

    sudo update-alternatives --auto docker

> Written with [StackEdit](https://stackedit.io/).
