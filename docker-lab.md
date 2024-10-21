# Docker Lab

- [Docker Lab](#docker-lab)
  - [Before starting](#before-starting)
  - [Task D1: Docker Hello world!](#task-d1-docker-hello-world)
  - [Task D2: Running Ubuntu](#task-d2-running-ubuntu)
  - [Task D3: Running an interactive shell on Ubuntu](#task-d3-running-an-interactive-shell-on-ubuntu)
  - [Task D4: Where are the images?](#task-d4-where-are-the-images)
  - [Task D5: Our first Dockerfile](#task-d5-our-first-dockerfile)
  - [Task D6: Containerizing an application](#task-d6-containerizing-an-application)
  - [Task D7: Creating a PostgreSQL container for development](#task-d7-creating-a-postgresql-container-for-development)
  - [Task D8: Docker compose](#task-d8-docker-compose)


## Before starting

Check the instructions on the [previous page](README.md).

## Task D1: Docker Hello world!

The following command will download the hello-world:latest image and run it:

```sh
docker run hello-world
```

Now, check the status of the container:

```sh
docker container ls -a

CONTAINER ID  IMAGE     COMMAND    CREATED     STATUS           PORTS   NAMES
763d61bd1f66  hello-world  "/hello"   15 seconds ago  Exited (0) 14 seconds ago       angry_mestorf
```

Note that the status "Exited": the container is not running anymore.

## Task D2: Running Ubuntu

The following command will download the ubuntu:latest image and run it:

```sh
docker run ubuntu
```

> **Question:** Why is there no output as before?


Now, check the status of the container:

```sh
docker container ls -a

CONTAINER ID  IMAGE     COMMAND    CREATED     STATUS           PORTS   NAMES
252e90596cd3  ubuntu    "/bin/bash"  10 seconds ago  Exited (0) 11 seconds ago       confident_hopper
```

Note that the status "Exited": the container is not running anymore.

Try the `ls` without `-a`:

```sh
docker container ls

CONTAINER ID  IMAGE     COMMAND    CREATED     STATUS           PORTS   NAMES
```

> **Question:** Why is the output empty?

> **Hint:** `docker ps` and `docker ps -a` were the old way of checking the containers.
> They still work and can be found in many tutorials everywhere.

## Task D3: Running an interactive shell on Ubuntu

```sh
docker run -it ubuntu bash

root@1b308e8aa1a3:/# 
```

If you executed the previous task, the `ubuntu:latest` image is already on your computer and will not be downloaded again. You can interact with the shell.

Keep this shell running and open a new terminal window. Then type the following command:

```sh
docker container ls -a

CONTAINER ID  IMAGE     COMMAND    CREATED     STATUS           PORTS   NAMES
1b308e8aa1a3  ubuntu    "bash"    3 minutes ago  Up 3 minutes             clever_bartik
252e90596cd3  ubuntu    "/bin/bash"  13 minutes ago  Exited (0) 13 minutes ago       confident_hopper
```

Compare the two Ubuntu containers: one is still running.

Get back to the previous terminal and type `exit`:

```sh
root@1b308e8aa1a3:/# exit
```

Then, recheck the containers:

```sh
docker container ls -a   

CONTAINER ID  IMAGE     COMMAND    CREATED     STATUS           PORTS   NAMES
1b308e8aa1a3  ubuntu    "bash"    6 minutes ago  Exited (0) 11 seconds ago       clever_bartik
252e90596cd3  ubuntu    "/bin/bash"  17 minutes ago  Exited (0) 17 minutes ago       confident_hopper
```

They both exited.


## Task D4: Where are the images?

Try the command below:

```sh
docker image ls

REPOSITORY  TAG    IMAGE ID    CREATED    SIZE
ubuntu    latest  e4c58958181a  3 weeks ago  77.8MB
hello-world  latest  9c7a54a9a43c  6 months ago  13.3kB
```

## Task D5: Our first Dockerfile

Create a folder named `msip-docker` on your computer.
In this folder, create a file named `Dockerfile` (the first letter must be a capital `D`) with the following content:

```
FROM ubuntu:20.04

CMD ["echo", "Hello", "World!"]
```

**Did you note that we used a specific version this time instead of `latest`?**

Open a terminal in this folder and build the image:

```sh
docker build . -t msip-hello
docker build . -t msip-hello:1.0
```

Next, check the images:

```sh
docker image ls

REPOSITORY  TAG    IMAGE ID    CREATED    SIZE
...
msip-hello  1.0    670fcf3ec158  4 weeks ago  72.8MB
msip-hello  latest  670fcf3ec158  4 weeks ago  72.8MB
```

We used a `tag` to specify the version. If a tag is not informed, `latest` is assumed.

Finally, run a container based on your new image:

```sh
docker run msip-hello
```


## Task D6: Containerizing an application

Clone the Docker "getting started" app and go to the `app` folder:

```sh
git clone https://github.com/docker/getting-started.git
cd getting-started/app
```

Create a `Dockerfile` in this folder with the following content:

```sh
FROM node:16-alpine
WORKDIR /app
COPY . .
RUN yarn install --production
CMD ["node", "src/index.js"]
```

Build and run the app:

```sh
docker build -t node-app .
docker run -p 3000:3000 node-app
```

Open your browser at [http://localhost:3000/](http://localhost:3000/).

> **Question:** Did you notice the application keeps the container running?

Stop the application with `Ctrl + C`. Then try the following command:

```sh
docker run -p 4000:3000 node-app
```

Open your browser at [http://localhost:4000/](http://localhost:4000/).

> **Question:** What happened here?

One more time, stop the application with `Ctrl + C` and try the following command:

```sh
docker run -d -p 4000:3000 node-app
```

Open your browser at [http://localhost:4000/](http://localhost:4000/). The container is now running as a background process. You can see, stop, and remove it with the following commands:

```sh
docker container ls
docker stop <container-id>
docker rm <container-id>
```

## Task D7: Creating a PostgreSQL container for development

Run the following commands in sequence:

```sh
docker pull postgres
docker pull dpage/pgadmin4
docker image ls
docker network create msip-network
docker network ls
docker run --name msip-postgres --network=msip-network -e "POSTGRES_PASSWORD=postgres" -d postgres:latest
docker run --name msip-pgadmin --network=msip-network -p 8100:80 -e "PGADMIN_DEFAULT_EMAIL=admin@admin.com" -e "PGADMIN_DEFAULT_PASSWORD=postgres" -d dpage/pgadmin4:latest
docker container ls
```

> **Hint:** Note that the new containers do not have random names anymore.

Open [http://localhost:8100/](http://localhost:8100/) on your preferred browser and proceed with the setup below:

![pgAdmin setup](images/pgadmin-setup.png)

Save the changes and connect to the server. You should now have access to the database running in the container.


## Task D8: Docker compose

Let's do some clean up:

```sh
docker stop msip-pgadmin msip-postgres
docker rm msip-pgadmin msip-postgres
docker network rm msip-network
```

In your `msip-docker` folder, create a file named `docker compose.yml` (no uppercase `D` this time) with the following content:

```version: '3'
services:
  msip-postgres:
    image: postgres:latest
    container_name: msip-postgres
    environment:
      POSTGRES_PASSWORD: postgres
    networks:
      - msip-network
  msip-pgadmin:
    image: dpage/pgadmin4:latest
    container_name: msip-pgadmin
    environment:
      PGADMIN_DEFAULT_EMAIL: admin@admin.com
      PGADMIN_DEFAULT_PASSWORD: postgres
    ports:
      - 8100:80
    networks:
      - msip-network
networks:
  msip-network:
    driver: bridge
```

Run the following command:

```sh
docker compose up
```

The containers will start as described. It's more straightforward than running Docker commands manually, isn't it?

Stop the process with `Ctrl + C` and run it in the "detach" mode:

```sh
docker compose up -d
```

You can stop all those containers with:

```sh
docker compose down
```
