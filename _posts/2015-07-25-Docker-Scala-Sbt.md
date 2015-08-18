---
date: 3012-08-23 09:14:00 +1000
layout: post
title: Dockerizing your Scala apps with sbt-docker
---

[Docker](https://www.docker.com) is the new hotness for app deployment -- and
for good reason.  It seems all the infrastructure providers are supporting it,
as is Mesos, etc.  This is a guide to dockerizing your Scala apps using [sbt-
docker](https://github.com/marcuslonnberg/sbt-docker) as well as setting up a
dev environment for Docker on OSX.  Also, hopefully this guide helps shed more light on SBT, which can be obtuse :)

I'm no Docker expert, but for the last few months my last company has been dockerizing their Scala apps onto an AWS / Mesos / Marathon-based infrastructure.  I also just wrapped up dockerizing my main OSS project, the [Spark Job Server](http://github.com/spark-jobserver/spark-jobserver), as well as helping with docker stuff for a big data training this month.

## Why Docker?

It's the infrastructure, stupid.   Mesos - Mesosphere DCOS - automatic scale up/down - rolling restarts - immutable infrastructure ...

as well as a Github/Git-like model of image management.

Docker is like Git for image management.  Once you moved to git, you didn't want to go back to subversion/perforce.

## Docker on OSX: My Setup

(Feel free to skip this section if you are on Linux)  The easiest way to get started with Docker on OSX is using [boot2docker](http://docs.docker.com/mac/step_one/).  One good guide to set everything up is [here](http://viget.com/extend/how-to-use-docker-on-os-x-the-missing-guide).  However, I found it easier to work with a real VM in terms of doing file/folder sharing, port forwarding, etc. -- it was easier to have control over these aspects.

In brief:

* Install VirtualBox
* Download and use an Ubuntu (14.04 or 14.10 server recommended as of this writing) OVA or VM image
* Set up docker - note the apt package is `docker-engine`, not `docker.io`, but better to use the script from docker.com:  `wget -qO- https://get.docker.com/ | sh`
* Configure a host-only network on the VM with static IP address
* Start docker daemon with TCP bindings instead of the default `/var/lib/docker.sock` binding: 
        sudo echo 'DOCKER_OPTS="-H 0.0.0.0:2375"' >>/etc/default/docker
        sudo service docker restart
* Install docker locally on OSX using `brew install docker`
* Configure `DOCKER_HOST` on OSX to point at your static IP:
        export DOCKER_HOST=tcp://192.168.56.10:2375

This setup will let you use the Docker daemon in your VM to create the containers, but drive it automatically from any process on OSX itself which calls `docker build` locally.

## sbt-docker

You could create your own Dockerfile and call docker manually to create the containers, but there are some reasons to use a tool integrated with SBT (assuming you use SBT for Scala development):

* Automatically pull in dependencies, or the correct path/name of your target jars
* Incorporate a docker image push into your release process via sbt-release

### Basic Options

#### ADD

#### ENV

The ENV directive in a Dockerfile sets up an environment variable at both container runtime as well as Docker image build time (ie the RUN commands can use it).  These are especially useful for reusing vars in both places.  An example is, for Spark Job Server, the SPARK_VERSION can be used both at docker build time and by the runtime scripts.

#### EXPOSE

### Run and Volume directives

Use short clear paths -- since the container has its own filesystem namespace, there is no reason to avoid short, clear paths, such as `/database`, `/logs`, `/spark`, etc.

If you need lots of RUN commands to set up your container, break them up into multiple RUN commands.  Stacking a long list of commands into a giant RUN makes debugging difficult and the entire list has to be re-run;  breaking them up means Docker can cache results of individual commands and only re-run what is needed.

### Pushing and Releasing

NOTE: you need to do `docker login` first before pushing.

## General tips on Dockerizing

Docker has [best practices](https://docs.docker.com/articles/dockerfile_best-practices/) for writing Dockerfiles.  Some things to think about:

* Modularity - each docker image is meant to be small and contain only one process.  If you find yourself stuffing tons of services into one container, consider breaking them apart.
* Ease to start - if possible, have `docker run` start everything automatically.  Users can configure the service using `-e` to override environment variables or command line arguments.

## Dockerizing Scala/JVM Apps and Spark

- Use a lightweight base Java image
- Consider using a Scala base image and not including Scala standard lib - iterate much faster!
- For an example of dockerizing Spark apps, have a look at my [Spark Job Server container](https://hub.docker.com/r/velvia/spark-jobserver/) on Dockerhub...
    + One problem with Spark is that there are so many versions ... it can be compiled for Hadoop 1.x, 2.4, 2.6, CDH; with or without Mesos; Spark 2.10 or 2.11. 
    + I solve this by using a base Java7 (Sun) image with Mesos, then using environment vars to control what Spark distribution to download and install
    + Meantime, in your Spark app, do not package Spark dependencies themselves nor Scala in the final assembly.  The details of this can be found in the Spark Job Server build files (`Build.scala`).

Happy dockerizing!