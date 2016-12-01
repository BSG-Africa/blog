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
Cluster management and orchestration using Docker is done by Docker Engines running in swarm mode. A swarm is a cluster of Docker Engines where you deploy services. When you run Docker Engine outside of swarm mode, you execute container commands. When you run the Engine in swarm mode, you orchestrate services.

##### Node

##### Services and tasks


Read more [here](https://docs.docker.com/engine/swarm/key-concepts/).

#### Plan your swarm
Start by planning how many nodes will be in your swarm, and which node will be the swarm master. The swarm master takes care of the orchestration of the whole swarm, and is also a worker node. First install Docker engine on each node, including the swarm master. 

##### Create VM's in Azure
First set up your virtual machines. An Azure virtual machine scale set works well for this, and includes a load balancer. For this example I chose to create a scale set with Ubuntu 14.04.5 LTS. After creating the scale set use the Azure Portal to find the IP address of the scale set load balancer and also examine the _Inbound NAT rules_ to determine the ports targeting port 22, for SSH.

SSH into your first VM and [install](https://docs.docker.com/engine/installation/linux/ubuntulinux/) Docker:

``` bash
$ sudo apt update
$ sudo apt install apt-transport-https ca-certificates
$ sudo apt-key adv \
               --keyserver hkp://ha.pool.sks-keyservers.net:80 \
               --recv-keys 58118E89F3A912897C070ADBF76221572C52609D
$ echo deb https://apt.dockerproject.org/repo ubuntu-trusty main | sudo tee /etc/apt/sources.list.d/docker.list
$ sudo apt update
$ sudo apt install linux-image-extra-$(uname -r) linux-image-extra-virtual
$ sudo apt install docker-engine
```

Start the Docker daemon:

``` bash 
$ sudo service docker start
```

Check that Docker has been correctly installed:

``` bash
$ sudo docker run hello-world
```

Repeat these steps on each of the VM's in the scale set. Each worker node in your swarm requires its own instance of Docker Engine.

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

With a storage solution in place we are now ready to configure our services.

#### Configure services
##### Overlay network
To ensure that the tasks in the swarm can communicate with one another, and to provide isolation from other tasks managed by the swarm, create an [overlay network](https://docs.docker.com/engine/swarm/networking/) specific to the TeamCity application. This will allow the server and agent containers to communicate with each other. Set up the network on the swarm manager:

``` bash
$ docker network create --driver=overlay teamcity_network
```

##### TeamCity server
We can now deploy the TeamCity server using the Docker image [supplied](https://hub.docker.com/u/jetbrains/) by Jetbrains. Run:

``` bash
$ docker service create --name teamcity-server --replicas 1 --network teamcity_network --mount type=volume,source=teamcity_datadir,destination=/data/teamcity_server/datadir,volume-label="teamcity" --mount type=volume,source=teamcity_logsdir,destination=/opt/teamcity/logs,volume-label="teamcity" --publish 8111:8111 jetbrains/teamcity-server
```

The above command does several things. It creates a new service running the `jetbrains/teamcity-server` image. There is one instance of the service and it is part of the `teamcity_network` network. Our two named datavolumes, backed by an Azure storage account, are mounted by the service and labelled, and lastly port 8111 is exposed.

##### TeamCity agents
The TeamCity server requires build agents to perform its tasks. Let's deploy three replicas of the TeamCity agent Docker image:

``` bash
$ docker service create --name teamcity-agent --replicas 3 --network teamcity_network --mount type=volume,destination=/data/teamcity_agent/conf --mount type=bind,source=/var/run/docker.sock,destination=/var/run/docker.sock --env SERVER_URL=52.178.44.136:8111 jetbrains/teamcity-agent
```

In this command we mount an anonymous datavolume for `/data/teamcity_agent/conf` by not specifying a `source`. This ensures that each task gets its own unique volume, and that each volume is removed when its task completes. This means that agent configuration is transient, and especially the authorisation on the server is not persisted.
To make the Docker daemon available on the agents, `/var/run/docker.sock` is mounted. This is our only option because the `--privileged` flag is not available in swarm mode.
The SERVER\_URL environment variable is set to the TeamCity server's public URL so that the agent knows where to register itself.

#### Inspecting your swarm

#### Complete TeamCity installation
<!--
1. SQL DB
2. JDBC Driver upload
-->

<!-- Footnote definitions -->
<!-- TODO : Be consistent: Inline links, or footnotes -->
[^swarm]: https://docs.docker.com/engine/swarm/key-concepts/
