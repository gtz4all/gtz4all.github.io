---
title: "Building a Hugo Docker Containers"
date: 2021-06-04T11:29:58-04:00
description: "Building a Hugo Docker Container using Multi-Stage Dockerfile"
---
## Building a Docker Image
#### Hugo Multi-Stage Dockerfile

Multi-stage builds can use multiple FROM statements in the Dockerfile. Each FROM instruction can use its own base image. Any artifacts from each FROM can be copied to another stage. The last FROM statement is used as final image. Everything else is destroyed. 

```sh
############################################
# Multi Stage- Hugo Static Page 
############################################

## Stage 1 - name 'build'
FROM nginx:alpine as build

## Setting Arguments
ARG HUGO_VERSION="0.72.0"
ARG STATIC_PAGE=""

## Running commands inside container during build
RUN apk add --update wget
RUN wget --quiet "https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_${HUGO_VERSION}_Linux-64bit.tar.gz" && \
    tar xzf hugo_${HUGO_VERSION}_Linux-64bit.tar.gz && \
    rm -r hugo_${HUGO_VERSION}_Linux-64bit.tar.gz && \
    mv hugo /usr/bin

## Copying local Hugo Static Site Directory to Container directory
COPY ./${STATIC_PAGE} /site
WORKDIR /site

## Building Site
RUN hugo

## Stage 2. 
FROM nginx:alpine
## copy /site/public folder created by the 'hugo' RUN command in Stage 1 to the nginx working directory.
COPY --from=build /site/public /usr/share/nginx/html

WORKDIR /usr/share/nginx/html
```
* Changin config.toml baseURL value

    Local Testing: export BASE_URL=/
    Gitlab Pages: export BASE_URL=https://netgtz.gitlab.io/
    Since the BASE_URL variable includes "/" using "|" as delimiter instead
    shell command: sed -i "s|BASE_URL|$BASE_URL|g" config.toml 

```sh
sed -i "s|BASE_URL|$BASE_URL|g" netgtz.gitlab.io/config.toml
```
* Building a Docker Image based on Dockerfile

```sh
docker build --build-arg STATIC_PAGE=netgtz.gitlab.io -t hugo .
```
* Running a container using the recently built image exposing container port 80 on local host port 80
```sh
docker run --rm -p 80:80 -d --name hugoapp hugo
```

## Running a Docker Image

* Commonly used Docker commands

    docker run [OPTIONS] IMAGE [COMMAND]
    docker run [--rm -p 80:80 -dit --name hugoapp] hugo [/bin/ash]

```sh
docker run --rm -p 80:80 -dit --name hugoapp hugo /bin/ash
 ```
 ###### OPTIONS
 Name|shorthand|Description
 --|--|--
--detach|-d|Run container in background and print container ID
--interactive|-i|Keep STDIN open even if not attached
--publish|-p|Publish local port to container port - -p 8080:80 - 8080=local_port, 80=container_port
--rm|--rm|Automatically remove the container when it exits
--tty|-t|Allocate a pseudo-TTY
--name|--name|Assign a name to the container

* To attach to an already running container:
```sh
## ubuntu - /bin/bash, Alpine - /bin/ash
docker exec -it hugoapp /bin/ash
```

#### Run the container in deattached (-d), interactive (-i) , terminal(-t) node
* this will allow the ducker to continue running /bin/bash
```sh
docker run -it -d IMAGE_NAME bin/bash
```

###### Create Multiple Directories ( -p)
```sh
 mkdir -p sbox/docker
```
