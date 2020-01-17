---
layout: post
title: "Docker: the beginner's cheatsheet"
image: "/images/docker-beginners-cheatsheet/containers.jpg"
published: true
---

In the past few months I have started working with Docker. Initially I found that there was a lot to remember in terms of Docker's key concepts and terminology, as well as all of the CLI commands and what each of them do. 

I have created this cheatsheet to help with remembering the key Docker concepts and terminology, as well as what I have found to be the most useful Docker CLI and Docker Compose commands. Keep in mind that the list of commands give here contains just the most useful commands for those who are starting out with Docker, and is by no means exhaustive. You can find a full list of Docker commands [here](https://docs.docker.com/engine/reference/commandline/cli/).

Also, I learned most of what is written in this article by doing the following Udemy course, (affiliate link) [Docker and Kubernetes: The Complete Guide](https://click.linksynergy.com/link?id=5FPU6FRy5w4&offerid=507388.1793828&type=2&murl=https%3A%2F%2Fwww.udemy.com%2Fcourse%2Fdocker-and-kubernetes-the-complete-guide%2F), which I recommend enroling on if you are looking to advance your knowledge of Docker further.


# Docker Concepts

These are the key concepts of Docker explained at a high level.

## Docker Server (aka Docker Daemon)

The [Docker Server](https://docs.docker.com/engine/docker-overview/#the-docker-daemon) (or Docker Daemon) is a process which runs in the background and is in charge of creating and manipulating docker objects such as containers and images. Commands entered via the Docker Client are sent to and handled by the Docker Server.

## Docker Client

The [Docker Client](https://docs.docker.com/engine/reference/commandline/cli/s) is a CLI tool through which commands are delivered to the Docker Server.

## Image

An [image](https://docs.docker.com/engine/reference/commandline/image/) is a Snapshot of a filesystem along with a startup command. Used as a blueprint for creating and running containers.

## Container

A [container](https://docs.docker.com/engine/reference/commandline/container/) is an instance of an image, created using that image as a blueprint. 

A container has access to a portion of its host system's resources - on Linux this is achieved using Namespacing and Control Groups, on other operating systems such as Windows and Mac a Linux virtual machine is installed alongside Docker, and containers are run inside of this virtual machine.

## Dockerfile

A file which contains a set of instructions for building a Docker image.

## Volume

A [volume](https://docs.docker.com/storage/volumes/) is a way of persisting data beyond the lifecycle of a Docker container. 

Essentially, a volume allows you to map a folder on the filesystem within a Docker Container to a folder external to the container. 

## Docker Compose

[Docker Compose](https://docs.docker.com/compose/) is used for working with and running multi-container Docker applications. 

You may wonder why it is necessary for an application to be split across multiple containers - it is because many applications are made up of multiple components which communicate over a network - eg. an API running on a Node.js server, which communicates with a database running on its own database server. Docker Compose makes it easy to run each of these components in their own container, and create and configure a network over which these components can communicate.

# Docker CLI commands

These are some of the most useful commands for working with the Docker CLI. 

## General commands

These are general commands that are useful when working with Docker:

1. `docker version`

    Display version information of Docker Client and Docker Server.

2. `docker ps`, `docker ps --all`

    List all running containers on your machine - this can be useful to obtain the ID of a running container. 
    
    The `--all` flag lists all containers which exist on your machine, as opposed to just running containers.

3. `docker cp [sourceFilePath] [containerId]:[destinationFilePath]` 

    Copy a file or folder from your local machine into a Docker container.


## Commands for working with containers

These are commands for creating and working with Docker containers.

1. `docker run [imageName]`

    The Docker Server will check your local image cache for the specified image. If the image is not found in the local image cache, it will then search for it on Docker Hub, and if found, download it and store it in your local image cache. Docker Server will then use the image as a sort of blueprint to create a container, and then start that container using the image's startup command.

    Note that running this command is equivalent to running the following 2 commands:

    `docker create [imageName]` 
    `docker start -a [containerId]`

2. `docker run [imageName] [command]`

    Same as the above, but allows us to overrun the startup command with the specified command.

3. `docker create [imageName]`

    Similar to `docker run [imageName]`, but will only create a container, but not start it.

4. `docker start -a [containerId]`

    Start a container which has already been created (and may have already been started and then stopped), by specifying its ID. 

    The startup command from the image used to create the container will be used (note that `docker start` **cannot** be used to override this startup command). The `-a` tells docker to 'attach' to the container, and print any output from the container to the terminal.

    Note that in the above, and whenever you are working with container IDs, you do not need to use the full id - just enough of it that the section of the ID supplied is unique amongst existing container IDs.

5.  `docker system prune`

    Clean up disk space on your machine by removing stopped containers, as well as other bits and pieces such as unused networks, dangling images, and the Docker build cache. You can verify that this has removed all stopped containers by running `docker ps --all` afterwards.

6. `docker logs [containerId]`

    Output all of the logs from a specified container to the terminal.

7. `docker stop [containerId]`

    Stop a container - Send a SIGTERM message to the process running inside the specified container (SIGTERM gives the processs a chance to clean itself up before stopping). If after 10 seconds, the process has not stopped, a SIGKILL will be ssent to the process to kill it.

8. `docker kill [containerId]`

    Kill a container - Send a SIGTERM message to the process running inside the specified container.

9. `docker exec -it [containerId] [command]`, `docker exec -it [containerId] sh`

    Execute a command inside the specific container. `-it`, is actually 2 flags:

    * `-i` wires up anything you type to the STDIN of the the container.
    * `-t` formats the output of the STDOUT from the container, in order to display it in a readable format in your terminal.

    Running this with `sh` as the commandmis often very useful. It will start a command line shell inside of the container, allowing you to then execute whatever commands you wish inside the container.

10. `docker run -it [imageName] sh`

    Create and start a container, running the `sh` command prompt as the startup command (you can then interact with the container via the command prompt).

11. `docker inspect [containerId]`

    Inspect a container to obtain low level information about it. This command outputs a large JSON object, and if we just want to inspect some specific property we can use the `-f` flag. 

    For example, `docker inspect -f '{{ json .Mounts }}' [containerId]`, will output information about the container's volumes.

## Commands for building images

These commands are concerned with building images using a Dockerfile:

1. `docker build [buildContext]`

    This command needs to be run in a folder containing a Dockerfile. It builds an image using the Dockerfile, using the specified folder path as the build context.

2. `docker build -t [yourDockerId]/[projectName]:[version] [buildContext]`

    Using the `-t` tag with docker build allows you to tag an image - as a convention, the above format is used for the tag.

3. `docker build -f [yourDockerFile] [buildContext]`

    Using the `-f` tag with docker build allows you specify a Dockerfile to user for the build (by default `./Dockerfile` is used).

# Docker Compose commands

These commands are concerned with creating and running containers as specified inside of a `docker-compose.yml` file.

1. `docker-compose up`

    Create and run any containers and networks specified in `docker-compose.yml`. By adding the `--build` flag, any rebuildable images that were used will be rebuilt before creating and running the containers.

2. `docker-compose down -v`

    This will stop and remove containers and networks created by `docker-compose up`. The `-v` flag specifies that volumes should also be removed.

3. `docker-compose run [service]`

    Create and run a container for the specified service within a `docker-compose` file

# Useful Resources

The following course has been very useful to me, and is a great way to get started with Docker (please note that the below is an affiliate link):

* [Docker and Kubernetes: The Complete Guide](https://click.linksynergy.com/link?id=5FPU6FRy5w4&offerid=507388.1793828&type=2&murl=https%3A%2F%2Fwww.udemy.com%2Fcourse%2Fdocker-and-kubernetes-the-complete-guide%2F) 