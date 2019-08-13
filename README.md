# Docker and Kubernetes: The complete guide

| # | Topic |
| - | ----- |
|1. |[Why use Docker](#Why-use-Docker)|
|2. |[What is Docker](#What-is-Docker)|
|3. |[Manipulate dockers with docker-cli](#Manipulate-dockers-with-docker-cli)|
|4. |[Build custom images](#Build-custom-images)|
|5. |[Make real project with Docker](#Make-real-project-with-Docker)|
|6. |[Docker compose with multiple local containers](#Docker-compose-with-multiple-local-containers)|
|7. |[Create a production-grade workflow](#Create-a-production-grade-workflow)|

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

:pushpin: **Run** a command in a **new** container

`docker run [OPTIONS] IMAGE [COMMAND]`

```bash
docker run busy-box echo hi there
```

And it is equal to

`docker create <image name>`: take the file system snapshot into the new container

and

`docker start -a <container id>`: execute the startup command, cannot override the startup command

```bash
docker create busy-box # will return the container id
docker start -a 912ewq68qwrkl241lqw # --attach(-a), Attach STDOUT/STDERR and forward signals
```

:pushpin: **Run** a command in a **running** container

`docker exec [OPTIONS] CONTAINER COMMAND`

- --interactive , -i: Keep STDIN open even if not attached
- --tty , -t :arrow-right: Allocate a pseudo-TTY

```bash
docker exec -it redis redis-cli
docker exec -it busy-box bash
```

:pushpin: *List* running containers

```bash
docker ps
# -a: shows all containers
```

:pushpin: *Build* Docker images from a Dockerfile and a 'context'

```bash
docker build -t 'kephin/myContainer' .
# -t: label the image
```

:pushpin: *display* the logs of a container

```bash
docker logs --follow kephin/myContainer
# --follow: follow the output in the logs of the program
```

:pushpin: *List* volumes

```bash
docker volume ls
```

:pushpin: *Remove* one or more *containers*

```bash
docker rm myContainer
```

:pushpin: *Remove* one or more *images*

```bash
docker rmi myImage
```

:pushpin: *Stop* one or more *containers*

`docker stop`: The main process inside the container will receive SIGTERM, and after a grace period, SIGKILL.

`docker kill`: The main process inside the container will receive SIGKILL.

```bash
docker stop myContainer
docker kill myContainer
```

:pushpin: **Remove** unused data

```bash
docker system prune -a
```

## Build custom images

Create a Dockerfile. The Dockerfile is a configuration to define how our container should behave

Basic flow of creating a Dockerfile:

1. Specify a base image:

    1. :mag: use cache image if we have
    2. *otherwise* download the alpine image from docker hub

2. Run some commands to install additional packages:

    1. :mag: use cache image if we have
    2. *otherwise* :arrow_up: get image from last step
    3. create a temporary container out of it
    4. run commands to install packages
    5. took the file system snapshot
    6. shut down the temporary container
    7. :arrow_down: get the image ready for the next step

3. Specify a command to run container startup

    1. :mag: use cache image if we have
    2. *otherwise* :arrow_up: get image from last step
    3. create a temporary container out of it
    4. tell the container it should run some commands when started
    5. shut down the temporary container
    6. get the image ready for the next step

```dockerfile
# use an existing docker image as base
FROM alphine
# download and install dependencies
RUN apk add --update redis
# tell the image what to do when it starts as a container
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

## Make real project with Docker

`alpine` is a term in docker world for a image that as small and compact as possible. Many popular repositories will offer an `alpine` version of their image. The `alpine` version of the image means that you're not going to get a bunch of pre-installed programs.

```dockerfile
FROM node:alpine

# copy the file from local machine to the container
# any following command will be executed relative to this path in the container
WORKDIR usr/app
# COPY [PATH to copy from local relative to build context] [PATH to copy to the container]
COPY ./ ./

RUN npm install
CMD ["node", "start"]
```

### Docker run with port mapping

:exclamation: We have to specify port mapping in the runtime, not inside of the Dockerfile

`docker run -p <route incoming request to this port on localhost>:<to this port inside the container> <image id/name>`

```bash
docker run -p 8080:8080 kephin/simpleweb
```

### Minimize cache busting and rebuilds

:bulb: We want to run `npm install` only when we change `package.json` during docker build. So if nothing is changed in `package.json`, docker will use the cache.

```dockerfile
FROM node:alpine

WORKDIR usr/app

COPY ./package.json ./
RUN npm install
COPY ./ ./

CMD ["node", "start"]
```

## Docker compose with multiple local containers

Options for connection multiple containers:

1. Use docker CLI networking features
2. Use docker-compose

The purpose of docker-compose is to essentially function as docker CLI but allow you to issue multiple commands much more quickly

- To start up **multiple containers** at the same time

- Automate some long-winded arguments we were passing to `docker run`

### Define `docker-compose.yml` file

Instead of running

```bash
docker build -t kephin/visits:latest .
docker run -p 8080:8080 kephin/visits
```

We create a `docker-compose.yml` file that contains all the options we'd normally pass to docker-cli

By defining services inside the `docker-compose.yml` file, docker-compose is going to automatically create those containers on the same network, and they'll have free access to communicate to each other.

```yml
version: '3'
services:
  redis-server:
    image: 'redis'
  visits-app:
    build: .
    ports:
      - '3000:8080'
```

:+1: Now redis-server and visit-app can communicate to each other automatically.

So inside our node application

```javascript
const express = require('express')
const redis = require('redis')

const client = redis.createClient({
  /*
  usually you'll need to put https://... as url

  but since we're making use of docker-compose, we can can connect to the other container simply by referring to its name
  */
  host: 'redis-server',
  port: 6379
})
// ...
```

### How to start up docker-compose using yml

If we want to run containers

```bash
# docker
docker run myImage
# docker-compose
docker-compose up
```

If we want to **re-build** the images

```bash
# docker
docker build .
docker run myImage
# docker-compose
docker-compose up --build
```

If we want to launch containers in background and then want to stop the container

```bash
# docker
docker run -d redis # -d: running at the background
docker stop neip122421
# docker-compose
docker-compose up -d # -d: running at the background
docker-compose down
```

### Container maintenance with compose - automatic container restarts

## Create a production-grade workflow

Create `Dockerfile.dev`

```dockerfile
FROM node:alpine

WORKDIR '/app'

COPY package.json .
RUN npm install
COPY . .

CMD ["npm", "start"]
```

Build the docker image

```bash
# need to specify the file name
# because by default it will look for Dockerfile
docker build -f Dockerfile.dev -t kephin/frontend .
```

Start the container

```bash
docker run -p 3000:3000 kephin/frontend
```

:raising_hand: We'll notice that it took quite long to build the image, and it's because we have **duplicated dependencies**

### :fire: Hot reload inside the container by docker volumes

With docker volumes, we essentially set up a **placeholder** inside of our container, so we no longer copy the entire `src` directory. Instead we're going to put a **reference** pointing back to our local machine.

So a **docker volume** can kind of be imagine as **port mapping**

| command | what does it do |
| ------- | --------------- |
|Port mapping|map a port from localhost(outside the container) to a port inside the container|
|Docker volume|map a folder from localhost(outside the container) to a folder inside the container|

```bash
docker run -p 3000:3000 -v $(pwd):/app kephin/frontend
```

But the command above will prompt you errors!

:hushed: Because there are `node_modules` folder inside our container but **NOT** in our local machine. So `node_modules` inside the container will be overwritten to nothing, which cause errors.

To fix this issue, we need to pass an additional `-v` flag with the only argument of the folder, which means we want it to be a placeholder for the folder that is inside the container. Don't map this folder. This is called `bookmarking` the volumes.

```bash
docker run -p 3000:3000 -v /app/node_modules -v $(pwd):/app kephin/frontend
```

### Shorthand with Docker Compose

Use `docker-compose` for all the configurations above.

```yml
version: '3'
services:
  web:
    build:
      context: .
      dockerfile: Dockerfile.dev
    ports:
      - '3000:3000'
    volumes:
      - '.:/app'
      - '/app/node_modules'
```

### Testing

:exclamation: But the issue of this approach is that it won't update the test whiling in development since we just build the image with the snapshot file system.

```bash
# create an image
docker build -f Dockerfile.env -t kephin/frontend .
# run the test
docker run -it kephin/frontend npm run test
```

:bulb: One approach is to

```bash
# after the container with volumes mapping setup is up
docker-compose up --build
# open another tab and
# we can run the test inside the container
docker exec -it kephin/frontend npm test
```

:bulb: Another approach is to add test service is docker-compose

```dockerfile
version: '3'
services:
  web:
    build:
      context: .
      dockerfile: Dockerfile.dev
    ports:
      - '3000:3000'
    volumes:
      - '.:/app'
      - '/app/node_modules'
  tests:
    build:
      context: .
      dockerfile: Dockerfile.dev
    volumes:
      - '.:/app'
      - '/app/node_modules'
    command: ["npm", "test"]
```
