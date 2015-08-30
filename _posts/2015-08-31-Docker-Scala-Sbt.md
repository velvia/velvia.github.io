---
date: 3012-08-23 09:14:00 +1000
layout: post
title: Dockerizing your Scala apps with sbt-docker
---

[Docker](https://www.docker.com) is the new hotness for app deployment -- and
for good reason.  It seems all the infrastructure providers are supporting it,
as is Mesos, etc.  This is a guide to dockerizing your Scala apps using [sbt-
docker](https://github.com/marcuslonnberg/sbt-docker) as well as setting up a
dev environment for Docker on OSX.  This guide will show how to use the power of Scala and [SBT](http://www.scala-sbt.org) to generate Docker configs and images, and you can never have enough blog posts to teach SBT :)

I'm no Docker expert, but for the last few months my last company has been dockerizing their Scala apps onto an AWS / Mesos / Marathon-based infrastructure.  I also just wrapped up dockerizing my main OSS project, the [Spark Job Server](http://github.com/spark-jobserver/spark-jobserver), as well as helping with docker stuff for a big data training this month.  You'll find the full code from the examples below in the [Build.scala](https://github.com/spark-jobserver/spark-jobserver/blob/master/project/Build.scala) file of the Job Server project. 

## Why Docker?

I'm sure this subject has been beaten to death, and there are some that feel like for JVM / Scala apps, the value of Docker is less because the JVM is already sort of a container environment.  So I'll keep this short and just share what value I've found from dockerization.

1. Right now, if you only had time to package your project or app one way, Docker is the way to go.
2. For people trying out your open source repo, `docker run` is as easy and more universal (arguably) than most alternatives.  No need to build anything.
3. Let's say you want to package a whole bunch of open source projects together for people to try out -- for our training we had to put together Cassandra, Kafka, Spark, and more.  Throw them all into one container for convenience.
4. I personally find the port and file namespacing helps.  It minimizes configuration and it's nice to know your database path is at `/database`.

That said, getting Docker setup on your dev laptop is nontrivial (unless you run Linux), and debugging Docker apps is just not as easy as on a regular box.

## Docker on OSX: My Setup

**NOTE**: Since I wrote the section below, [Docker Machine](https://docs.docker.com/machine/) came out and will supercede boot2docker.

(Feel free to skip this section if you are on Linux)  The easiest way to get started with Docker on OSX is using [boot2docker](http://docs.docker.com/mac/step_one/).  One good guide to set everything up is [here](http://viget.com/extend/how-to-use-docker-on-os-x-the-missing-guide).  However, I found it easier to work with a real VM in terms of doing file/folder sharing, port forwarding, etc. -- it was easier to have control over these aspects.

In brief:

* Install VirtualBox
* Download and use an Ubuntu (14.04 or 14.10 server recommended as of this writing) OVA or VM image
* Set up docker - note the apt package is `docker-engine`, not `docker.io`, but better to use the script from docker.com:  `wget -qO- https://get.docker.com/ | sh`
* Configure a host-only network on the VM with static IP address
* Start docker daemon with TCP bindings instead of the default `/var/lib/docker.sock` binding:

    ```bash 
    sudo echo 'DOCKER_OPTS="-H 0.0.0.0:2375"' >>/etc/default/docker
    sudo service docker restart
    ```

* Install docker locally on OSX using `brew install docker`
* Configure `DOCKER_HOST` on OSX to point at your static IP:
        export DOCKER_HOST=tcp://192.168.56.10:2375

This setup will let you use the Docker daemon in your VM to create the containers, but drive it automatically from any process on OSX itself which calls `docker build` locally.

## sbt-docker

You could create your own Dockerfile and call docker manually to create the containers (using `docker build -t myuser/myrepo .`), but there are some reasons to use a tool integrated with SBT (assuming you use SBT for Scala development):

* Automatically pull in dependencies, or the correct path/name of your target jars
* Ensure build dependencies match your runtime environment.
    - For example, for Spark Job Server, it must be built against a specific version of Spark (and Hadoop), and it's important to include a matching Spark distribution in the container itself.
* Use the full power of Scala, including build variables, to generate your Dockerfile
* Incorporate a docker image push into your release process via sbt-release

To get started, add this line to your `project/plugins.sbt`:

    addSbtPlugin("se.marcuslonnberg" % "sbt-docker" % "1.2.0")

Then, add a section to your `build.sbt` file:

```scala
import sbtdocker.DockerKeys._

lazy val dockerSettings = Seq(
    // things the docker file generation depends on are listed here
    dockerfile in docker := {
        // any vals to be declared here
        new sbtdocker.mutable.Dockerfile {
            <<docker commands>>
        }
    }
)  
```

The next section flushes out some of the docker commands per `sbt-docker` syntax.

### Basic Options

#### FROM

Every Docker container inherits from a base container that has a Linux distro, and for Scala apps, some flavor of the JVM.  You will probably want to add a command like `from("java:7")` here.  For Spark Job Server, we also wanted Mesos, so I put `from("ottoops/mesos-java7")` here.  Beware that the standard Java7 base container is OpenJDK, and if you require Sun's JRE, you need to use one of the countless variants. `ottoops` uses Sun JRE and is fairly lightweight.

#### ADD/COPY

`ADD` is the standard command in a Dockerfile to add a file to your docker container.  You could add entire directories.  Note that every `ADD` is treated as a separate layer by Docker, meaning that if you were to rebuild your container, Docker will skip the ADDs whose source has not changed.

The sbt-docker syntax for an add is like this:

```scala
  add(baseDirectory(_ / "config" / "docker.conf").value, file("app/docker.conf"))
```

(NOTE: the above assumes SBT >= 0.13.0)  The first parameter to add is an SBT path.  `baseDirectory` is a function which returns a file path relative to the base directory where `build.sbt` is located.  The second parameter is the container file path.

Per [Docker Best Practices](https://docs.docker.com/articles/dockerfile_best-practices/), it is better to use COPY than ADD, so the above becomes:

```scala
  copy(baseDirectory(_ / "config" / "docker.conf").value, file("app/docker.conf"))
```

#### Adding your Scala assembly jar

Thus far, the examples are things you could have done in your Dockerfile directly, but here is an example of using the power of SBT: adding your assembly jar.  Let's say you want to add an assembly jar when you run `docker` in sbt, and the assembly jar is from another project.  Here is the code from the job server:

```scala
lazy val dockerSettings = Seq(
    // Make the docker task depend on the assembly task, which generates a fat JAR file
    docker <<= (docker dependsOn (AssemblyKeys.assembly in jobServerExtras)),
    dockerfile in docker := {
      val artifact = (AssemblyKeys.outputPath in AssemblyKeys.assembly in jobServerExtras).value
      val artifactTargetPath = s"/app/${artifact.name}"
      new sbtdocker.mutable.Dockerfile {
        from("ottoops/mesos-java7")
        copy(artifact, artifactTargetPath)
      }
    }
)
```

Let's break this down:

* `docker <<= (docker dependsOn ...` tells SBT that the docker task (to build a container) depends on the output of the assembly task, from project `jobServerExtras`.
* `val artifact = ` line finds the full path of the assembly JAR from that project
* `val artifactTargetPath` creates the target (inside container) path 

#### EXPOSE

Exposing a port is super easy:

```scala
  expose(8090)
  expose(9999)    // for JMX  
```

Note that I like to expose a JMX port for app debugging, and you can configure JMX to use only one port, which makes EC2 and Mesos operation much easier.

#### ENV

The ENV directive in a Dockerfile sets up an environment variable at both container runtime as well as Docker image build time (ie the RUN commands can use it).  These are especially useful for reusing vars in both places.  An example is, for Spark Job Server, the SPARK_VERSION can be used both at docker build time and by the runtime scripts:

```scala
    env("SPARK_BUILD", s"spark-${sparkVersion}-bin-hadoop2.4")
    runRaw("""wget http://d3kbcqa49mib13.cloudfront.net/$SPARK_BUILD.tgz && \
              tar -xvf $SPARK_BUILD.tgz && \
              mv $SPARK_BUILD /spark && \
              rm $SPARK_BUILD.tgz
           """)
```

The above links the sparkVersion scala variable, which is also used during the jobserver SBT build to fetch dependencies to compile against, to the version of Spark downloaded in the RUN command when the container is built.  Note how scala code could be used to generate parts of the Dockerfile.

#### RUN

`RUN` executes shell commands while building the container.  It is typically used to install packages.  The `sbt-docker` syntax offers at least two variants: `runRaw` which you saw above to execute any text as a shell command (including env var substitution), and `run()` which allows you to input each arg separately:

```scala
  run("mkdir", "-p", "/database")
```

Use short clear paths -- since the container has its own filesystem namespace, there is no reason to avoid short, clear paths, such as `/database`, `/logs`, `/spark`, etc.

If you need lots of RUN commands to set up your container, break them up into multiple RUN commands.  Stacking a long list of commands into a giant RUN makes debugging difficult and the entire list has to be re-run;  breaking them up means Docker can cache results of individual commands and only re-run what is needed, resulting in huge speedups (especially when many of the RUNs tend to do things like `apt-get install`).

#### Volume

Docker volumes can be created to persist data beyond the lifetime of a container session, and to make it easier to pass files back and forth.  For example, the Spark Job Server uses a volume to persist database state so that it is preserved for subsequent `docker run` invocations.  Be sure to create the directory using a run command.

```scala
    // Use a volume to persist database between container invocations
    run("mkdir", "-p", "/database")
    volume("/database")
```

#### EntryPoint

Finally, you probably want to specify the script or command to start your app:

```scala
    entryPoint("app/server_start.sh")
```

### Pushing and Releasing

`sbt dockerPush` will build and push the image for you.  This is just a convenience - it is equivalent to doing `sbt docker`, which builds the image in your docker daemon in your Linux VM, and invoking `docker push <image-id>` from the command line.

You can see the results at the [Spark Job Server container](https://hub.docker.com/r/velvia/spark-jobserver/) on Dockerhub...

NOTE: you need to do `docker login` first before pushing.

Note that `sbt-docker` generates a Dockerfile in `target/docker/Dockerfile`.

## General tips on Dockerizing

Docker has [best practices](https://docs.docker.com/articles/dockerfile_best-practices/) for writing Dockerfiles.  Some things to think about:

* Modularity - each docker image is meant to be small and contain only one process.  If you find yourself stuffing tons of services into one container, consider breaking them apart.
* Consider using a Scala base image and not including Scala standard lib - iterate much faster!
* Ease to start - if possible, have `docker run` start everything automatically.  Users can configure the service using `-e` to override environment variables or command line arguments.

Happy dockerizing!