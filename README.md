# Docker and Kubernetes: The complete guide

| # | Topic |
| - | ----- |
|1. |[Why use Docker](#Why-use-Docker)|
|2. |[What is Docker](#What-is-Docker)|

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
