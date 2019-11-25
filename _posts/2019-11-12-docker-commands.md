---
layout: post
title: "Docker - the beginner cheatsheet"
image: ""
published: false
---

I have recently started working with Docker, but I often have trouble remembering the key concepts behind Docker, as well as all of the CLI commands and what they do. I have created this article as a cheatsheet for remembering what I feel are the key Docker concepts, as well as the most useful Docker CLI and Docker Compose commands.

# Docker Concepts

These are the key concepts of Docker explained at a high level.

## Docker Server (or Docker Daemon)

This is a process which runs in the background and is in charge of creating and manipulating docker objects such as containers and images. Commands entered via the Docker Client are sent to and handled by the Docker Server.

## Docker Client

CLI tool through which commands are delivered to the Docker Server.

## Image

Snapshot of a filesystem along with a startup command. Used as a blueprint for creating and running containers.

## Container

An instance of an image, created using that image as a blueprint. 

Has access to a portion of its host system's resources - on Linux this is achieved using Namespacing and Control Groups, on other operating systems such as Windows and Mac a Linux virtual machine is installed with docker, and containers are run inside of this virtual machine.

## Dockerfile

A file which contains a set of instructions for building a Docker image.

## Volume

Volumes are a way of persisting data beyond the lifecycle of a Docker container. 

Essentially, a volume allows you to map a folder on the filesystem within a Docker Container to a folder external to the container. 

## Docker Compose

Docker Compose is used for working with and running multi-container Docker applications. 

Why is it necessary for an application to be split across multiple containers?

Many applications are made up of multiple components which communicate over a network - eg. an API running on a Node.js server, which communicates with a database running on its own database server. Docker Compose makes it easy to run each of these components in their own container, and create and configure a network over which these components can communicate.

# Docker CLI commands

These are some of the most useful commands to remember when working with the Docker CLI. Note that when working with container IDs, you do not need to use the full id, just enough of it the the section of the ID supplied is unique amongst existing container IDs.

## General commands

These are general commands that are useful when working with Docker:

1. `docker version`

    Display version information of Docker Client and Docker Server.

2. `docker ps`, `docker ps --all`

    List all running containers on your machine - this can be useful to obtain the ID of a running container. The `--all` flag lists all containers which exist on your machine, as opposed to just running containers.

3. `docker cp [sourceFilePath] [containerId]:[destinationFilePath]` 

    Copy a file or folder from your local machine into a Docker container.


## Commands for working with containers

These are commands for creating and working with Docker containers.

1. `docker run [imageName]`

    Docker server will check the local image cache for the specified image. If the image is not found in the local image cache, search for it on Docker Hub and if found download it and store it in your local image cache. Docker server will now use the image (as a sort of blueprint) to create a container, and start that container using the image's startup command.

2. `docker run [imageName] [command]`

    Same as the above, but allows us to overrun the startup command with the specified one.

3. `docker create [imageName]`

    Similar to `docker run [imageName]`, but will only create, not start, the image.

6. `docker start -a [containerId]`

    Start a container which has already been created (and may have already been started and then stopped), by specifying its ID. 

    The startup command from the image used to create the container will be used (note that `docker start` **cannot** be used to override this startup command). The `-a` tells docker to 'attach' to the container, and print any output from the container to the terminal.

7.  `docker system prune`

    Clean up disk space on your machine by removing stopped containers, as well as other bits and pieces such as unused networks, dangling images, and the Docker build cache. You can verify that this has removed all stopped containers by running `docker ps --all`.

8. `docker logs [containerId]`

    Output all of the logs from a specified container to the terminal.

9. `docker stop [containerId]`

    Stop a container - Send a SIGTERM message to the process running inside the specified container (SIGTERM gives the processs a chance to clean itself up before stopping). If after 10 seconds, the process has not stopped, a SIGKILL will be ssent to the process to kill it.

9. `docker kill [containerId]`

    Kill a container - Send a SIGTERM message to the process running inside the specified container.

10. `docker exec -it [containerId] [command]`, `docker exec -it [containerId] sh`

    Execute a command inside the specific container. `-it`, is actually 2 flags:

    * `-i` wires up anything you type to the STDIN of the the container.
    * `-t` formats the output of the STDOUT from the container, in order to display it in a readable format in your terminal.

    Running this with `sh` as the commandmis often very useful. It will start a command line shell inside of the container, allowing you to then execute whatever commands you wish inside the container.

12. `docker run -it [imageName] sh`

    Create and start a container, starting up a command prompt as the startup command.

13. `docker inspect [containerId]`

    Inspect a container to obtain low level information about it. This command outputs a large JSON object, and if we just want to inspect some specific property we can use the `-f` flag. 

    For example, `docker inspect -f '{{ json .Mounts }}' [containerId]`, will output information about the container's volumes.

## Commands for building images

These commands are concerned with building images using a Dockerfile:

1. `docker build [buildContext]`

    This command needs to be run in a folder containing a Dockerfile. It builds an image using the Dockerfile, using the specified folder path as the build context.

2. `docker build -t [yourDockerId]/[projectName]:[version]`

    Using the `-t` tag with docker build allows you to tag an image - as a convention, the above format is used for the tag.

# Docker Compose commands

These commands are concerned with creating and running containers as specified inside of a `docker-compose.yml` file.

1. `docker-compose up`

    Create and run any containers and networks specified in `docker-compose.yml`. By adding the `--build` flag, any rebuildable images that were used will be rebuilt before creating and running the containers.

2. `docker-compose down -v`

    This will stop and remove containers and networks created by `docker-compose up`. The `-v` flag specifies that volumes should also be removed.

3. `docker-compose run [service]`

    Create and run a container for the specified service within a `docker-compose` file

# Useful Resources

The following course has been very useful to me, and is a great way to get started with Docker if you have no experience using it.

TODO: affiliate link?

* <https://www.udemy.com/course/docker-and-kubernetes-the-complete-guide/> 
