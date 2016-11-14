---
title: Set up TeamCity in a Docker Swarm
author: nieldewet
---

Since the release of Docker Engine v1.12.0 [swarm mode](https://docs.docker.com/engine/swarm/) is no longer a separate component project but is now part of the core functionality offered by Docker Engine. With your tasks running on a cluster and managed by the swarm you can rely on Docker to ensure that your tasks are always running, even if some of the nodes in the cluster go off-line. This is realy powerful and convenient, but requires a bit more thinking about your architecture and more configuration. As an example I will show how to set up TeamCity with swarm mode on more than one node running in Azure.<!--more-->

<!--
[TeamCity](https://www.jetbrains.com/teamcity/)
[Docker Hub](https://hub.docker.com/u/jetbrains/)
[swarm mode](https://docs.docker.com/engine/swarm/)
-->

<!--
    TODO : Figure out Azure setup, including load balancing[sic].
-->

#### Definitions

##### Swarm
swarm[^swarm]

##### Node
node[^swarm]

##### Services and tasks
services and tasks[^swarm]

#### Plan your swarm
<!--
1. Master: `swarm init`
2. Work nodes: Docker eng, network connection to master
3. Storage
-->
Start by planning how many nodes will be in your swarm, and which node will be the swarm master. The swarm master takes care of the orchestration of the whole swarm, and is also a worker node. First install Docker engine on each node, including the swarm master. 

##### Swarm master
Initialise the swarm master with the `swarm init` command:

``` bash
$ sudo docker swarm init
```

##### Workers
To instruct worker nodes to join the swarm, use the command in the output of the above command. For example, you may see:

``` text
Swarm initialized: current node (bvz81updecsj6wjz393c09vti) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join \
    --token SWMTKN-1-3pu6hszjas19xyp7ghgosyx9k8atbfcr8p2is99znpy26u2lkl-1awxwuwd3z9j1z3puu7rcgdbx \
    172.17.0.2:2377

    To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```

Run the `swarm join` command from the output on each worker node. It is also posible to add another node as a swarm master.

##### Storage
<!--
Q: How to assign cloud data volume to replicated services?
a) Multiple file shares on the storage account?
b) Using randon volume names for replicated services?
c) A combination of the above?
-->
Since you are delegating the responsibility of deciding on which node your containers will run to the swarm master, you need a way to ensure that the storage volume used by a task is always available to it, even if that task is moved to another node. If a task running on a particular node with a data volume mounted on the host is shifted to a different node, that data volume will stay behind and the task will loose access to it. This can be overcome by using a storage volume that is not limited to the host, like cloud data storage.

As an example, I will show how to set up your swarm to use Azure storage accounts. This involves two main steps, namely setting up the storage account on Azure, and setting up the storage driver as a daemon on each node.

On the Azure portal, start by creating a general purpose storage account. Once the storage account is deployed, create a File share by chosing the "Files" service and then adding a file share for each data volume. From the menu on the left under "Settings" choose "Access keys" and note the storage account name and the keys. We will use these to connect to the storage account from our nodes.

In order to access the Azure storage account using network file sharing protocols you need to install the [Docker Volume Driver for Azure File Storage](https://github.com/Azure/azurefile-dockervolumedriver) as a daemon on each node. I have created [a script](https://gist.github.com/nieldw/df5b672ffd70a88c341be24660d817e0) to handle the installation using upstart on hosts running Ubuntu 14.04 or earlier. A key step in the installation process is editing `/etc/default/azurefile-dockervolumedriver` with your Azure storage account credentials. This is also taken care of in the script. 

Once the storage account is in place and the drivers are installed we can create the necessary volumes. For the TeamCity server we need two volumes with corresponding file shares on the storage accounts: One for the TeamCity data directory, and one for the TeamCity logs directory. Run these commands to create the volumes on each node:

<!-- TODO : Is it necessary to run this on each node? -->
``` bash
$ docker volume create --name teamcity_datadir --driver azurefile --opt share=teamcityswarmdatadir
$ docker volume create --name teamcity_logsdir --driver azurefile --opt share=teamcityswarmlogs
```

<!-- TODO : Volumes for agents -->

With a storage solution in place we are now ready to configure our services.

#### Configure services
<!--
1. Overlay network
2. TeamCity server
3. TeamCity agent with Docker support
-->
To ensure that the tasks in the swarm can communicate with one another, and to provide isolation from other tasks managed by the swarm, create an [overlay network](https://docs.docker.com/engine/swarm/networking/) specific to the TeamCity application. This will allow the server and agent containers to communicate with each other. Set up the network on the swarm manager:

``` bash
$ docker network create --driver=overlay teamcity_network
```

#### Inspecting your swarm

#### Complete TeamCity installation
<!--
1. SQL DB
2. JDBC Driver upload
-->

<!-- Footnote definitions -->
<!-- TODO : Be consistent: Inline links, or footnotes -->
[^swarm]: https://docs.docker.com/engine/swarm/key-concepts/
