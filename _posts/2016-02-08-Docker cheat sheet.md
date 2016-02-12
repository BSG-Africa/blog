---
title: Docker cheat sheet
author: Rowan Titlestad
---

Here are a list of common docker commands we have been using on our project that involves the use of Docker 
<!--more--> 

| Action                 |      Docker Command| 
|------------------------|:------------------|
| Build an image |  `docker build -t <image tag> <path to Dockerfile>` | 
| Connect to a running container |    `docker exec -i -t <container id> bash`  |  
| Remove all dangling volumes | `docker-machine ssh default docker volume rm $(docker volume ls -q -f dangling=true)` |   
| Remove all stopped containers |    `docker-machine ssh default docker rm $(docker ps -a -q)`   |  
| Remove all untagged images | `docker-machine ssh default docker rmi $(docker images -q --filter "dangling=true" --no-trunc=true)` |   
| Run a container from an existing image |    `docker run <image name>`   |  
| View a list of all containers | `docker ps` |   
| View a list of all images |    `docker images`   |  
| View a list of all volumes | `docker volume ls` |   

