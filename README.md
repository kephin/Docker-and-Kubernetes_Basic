# Docker and Kubernetes: The complete guide

| # | Topic |
| - | ----- |
|1. |[Why use Docker](#Why-use-Docker)|
|2. |[What is Docker](#What-is-Docker)|
|3. |[Manipulate dockers with docker-cli](#Manipulate-dockers-with-docker-cli)|
|4. |[Build custom images](#Build-custom-images)|

## Why use Docker

Docker makes it really easy to install and run software without worrying about **setup** or **dependencies**

## What is Docker

Docker is a platform or ecosystem around creating and running containers

| Term | What does it do |
| ---- | --------------- |
| Images | Single file with all the dependencies and config required to run a program. Any time we talked about image, we're talking about a **file system snapshot** and a **startup command** |
| Containers | Instance of an image, which runs a program. The container is a program with its own **isolated set** of hardware resources. So container is NOT a physical construct that exists inside your computer. Instead a container is really a process or a set of processes that have a group of resources specially assigned to it |

Image when we run

```bash
docker run hello-world
```

that starts the `Docker Client`(`Docker CLI`). The `Docker CLI` is in charge of taking commands to communicate to `Docker server`. And it's the `Docker server` that is in charge of the heavy lifting.

The `Docker server` saw that we tried to start up a container using an image called `hello-world`. The first thing `Docker server` did is to see if it already had a local copy, called `Image cache`. If it didn't have the cached, then will reach out to `Docker hub` to download the images.

After that `Docker server` use this image to run the container.

### :question: What behind the scenes when taking an image and turn it into a container

First off, **kernel** will isolate a portion of the hard drive and make it available to just this container. So inside the specific group of resources, we have a portion of hard drive that has just the **file system snapshot** and nothing else. The startup command is then executed.

## Manipulate dockers with docker-cli

*Create* and *run* a container from an image, and we can optionally override the default command

`docker run <image name> <command>`

```bash
docker run busy-box echo hi there
```

And it is equal to

`docker create <image name>`: take the file system snapshot into the new container

plus

`docker start -a <container id>`: execute the startup command, cannot override the startup command

```bash
docker create busy-box # will return the container id
docker start -a 912ewq68qwrkl241lqw
```

*List* running containers

```bash
docker ps
# -a: shows all containers
```

*Build* Docker images from a Dockerfile and a 'context'

```bash
docker build -t 'kephin/myContainer' .
# -t: label the image
```

*Run* a docker container based on an image

```bash
docker run kephin/myContainer -it bash
# -it bash: run bash from within the container
```

*display* the logs of a container

```bash
docker logs --follow kephin/myContainer
# --follow: follow the output in the logs of the program
```

*List* volumes

```bash
docker volume ls
```

*Remove* on or more *containers*

```bash
docker rm myContainer
```

*Remove* one or more *images*

```bash
docker rmi myImage
```

*Stop* one or more *containers*

```bash
docker stop myContainer
docker kill myContainer
```

```bash
docker system prune -a
```

:star: Execute an additional command in a container

`docker exec -it <docker name> <command>`

```bash
docker exec -it redis redis-cli
docker exec -it busy-box sh
docker exec -it busy-box zsh
docker exec -it busy-box bash
```

```bash
# Execute Docker image
docker run busybox
# List Docker containers (running, all, all in quiet mode)
docker container ls
docker container ls --all
docker container ls -aq

docker exec -it busybox bash
```

## Build custom images

Create a Dockerfile. The Dockerfile is a configuration to define how our container should behave

Basic flow of creating a Dockerfile:

1. Specify a base image: download the alpine image
2. Run some commands to install additional packages:

    1. get image from last step
    2. create a temporary container out of it
    3. run commands to install packages
    4. took the file system snapshot
    5. shut down the temporary container
    6. get the image ready for the next step

3. Specify a command to run container startup

    1. get image from last step
    2. create a temporary container out of it
    3. tell the container it should run some commands when started
    4. shut down the temporary container
    5. get the image ready for the next step

```dockerfile
FROM alpine
RUN apk add --update redis
CMD ["redis-server"]
```

```bash
docker build .
```

### Tagging image

`docker build -t <docker-id>/<repo/project-name>:<version> <directory of the file to use for the build>`

```bash
docker build -t kephin/redis-server:latest .
## then you can just do
docker run kephin/redis-server
```
