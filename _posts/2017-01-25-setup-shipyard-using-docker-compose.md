---
title: Docker-Compose script for deploying Shipyard
author: jaisheelmistry
---

We have setup [shipyard](https://shipyard-project.com/) for monitoring docker containers for one of our clients projects. We however encountered that we could not use shipyards [automated deployment script](https://shipyard-project.com/docs/deploy/automated/) due to strict security restrictions. We however do have access to use dockers orchestration tool docker-compose. Below is the docker-compose.yml file that was derived from the manual docker run steps given [here](https://shipyard-project.com/docs/deploy/manual/)  <!--more-->

The script below will require you to set the `${HOSTNAME}` and `${IP_OF_HOST}` enviroment variables.
The `${IP_OF_HOST}` should be the ip address of the docker deamon this can be obtained by running `docker-machine ls`. 


    version: '2'
    services:
        shipyard-rethinkdb:
            image: rethinkdb
            restart: always
        shipyard-discovery:
            image: microbox/etcd
            restart: always
            ports:
                 - "4001:4001"
                 - "7001:7001"
            command: -name discovery
            depends_on:
                 - shipyard-rethinkdb
        shipyard-proxy:
            image: shipyard/docker-proxy
            hostname: ${HOSTNAME}
            volumes:
                 - "/var/run/docker.sock:/var/run/docker.sock"
            ports:
                 - "2375:2375"
            expose:
                 - "2375"
            depends_on:
                 - shipyard-discovery
        shipyard-swarm-manager:
            image: swarm
            restart: always
            command: manage --host tcp://0.0.0.0:3375 etcd://${IP_OF_HOST}:4001
            depends_on:
                - shipyard-proxy
        swarm-agent:
            image:  swarm
            restart: always
            command: join --addr ${IP_OF_HOST}:2375 etcd://${IP_OF_HOST}:4001
            depends_on:
                    - shipyard-swarm-manager
        shipyard-controller:
            image: shipyard/shipyard
            links:
            - shipyard-rethinkdb:rethinkdb
            - shipyard-swarm-manager:swarm
            ports:
                 - "8080:8080"
            command: server -d tcp://swarm:3375
            depends_on:
                    - swarm-agent

 

> Written with [StackEdit](https://stackedit.io/).