---
title: "Why Docker Images Need A Base Image"
last_modified_at: 2025-04-08T10:30:02-05:00
categories:
  - DevOps
tags:
  - Docker
  - DevOps
classes: wide
---

It seems strange that you need to provide a base image in a Dockerfile when building your custom docker image, even though 
Docker is run on a Linux operating system. Why do I need to specify a base image then?

Docker uses technics like `namespaces` and `cgroups` provided by Linux to isolate docker `containers` (running instances of docker `images`) from each other and from the host operating system.
Therefore the the Linux tools like the `bash shell`, a package manager like `apt-get`are **not available to your custom image** even though they might be available in the host.
You also wanna ensure that all libraries and dependencies your application needs to run are also available in your image. That is the reason you need a base image (a Linux distribution) to ensure that
your application has all required dependencies (install them using a docker command in your Dockerfile if not provided by the base image) and is able to run.
That way your docker image can run consistenly on any Linux distribution with a compatible kernel and hardware architecture.
