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

	brew install colima