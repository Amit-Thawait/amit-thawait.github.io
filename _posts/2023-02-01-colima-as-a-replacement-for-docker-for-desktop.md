---
title: Colima as a replacement for Docker for Desktop
layout: post
published: true
category: programming
tags: [docker, colima, containers]
comments: true
draft: true
---

### What is colima?

Colima (https://github.com/abiosoft/colima) means Containers in Lima (https://github.com/lima-vm/lima). Lima is linux virtual machines, typically on macOS, for running containerd.

Containerd is the container runtime of docker engine that used to get installed as part of Docker for mac.

Colima uses lima that runs containerd hence this is a drop in replacement of Docker for mac.

### Steps:
---

1) Uninstall "Docker for Mac" (Settings >> Uninstall)

   Go to Applications folder in mac and move Docker.app to bin.

2) Install colima using brew

   `brew install colima`

3) Install Docker

   `brew install docker`

   If there is any warning for docker already installed then run the below command:

   `brew link docker`

   Verify installation by entering `docker` in terminal and press enter.

4) Install docker-compose

   `brew install docker-compose`

   Verify docker-compose by entering

   `docker-compose version`

5) Start colima

   `colima start --cpu 4 --memory 12 --disk 100`

   You can choose for yourself how much amount of CPU, RAM and disk storage you would like to allocate in the above command.

   If the above command fails because of some network error then run the same command again.

   You can check its status by entering

   `colima status`

6) After uninstalling "Docker for Desktop", the tiny VM on your mac that contains all the volumes and images gets deleted. Hence you need to again build images for any of your application again or pull them again from docker hub. These images will get stored in VM instantiated by colima.

In case you face any issues like below

   <span class="text-red">
   => ERROR COPY target/build-info/* /build-info/
   -----
   failed to solve: lstat /var/lib/docker/tmp/buildkit-mount38899890/target/build-info: no such file or directory
   make: *** [docker-up] Error 17
   </span>

   then export the below environment variable in your terminal (as well as bashrc file or zsh profile

   `export DOCKER_BUILDKIT=0`