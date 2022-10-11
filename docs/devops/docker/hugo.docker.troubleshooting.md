---
title: "Working with Hugo Container"
date: 2021-06-04T11:29:58-04:00
description: "Creating Hugo Image with a Single Stage Dockerfile"
---
## Building The Hugo Docker Image
#### Hugo Single-Stage Dockerfile
We are just removing the second stage in order to take a closer look
```sh
FROM nginx:alpine as build

RUN apk add --update \
    wget

ARG HUGO_VERSION="0.72.0"
ARG STATIC_PAGE=""

RUN wget --quiet "https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_${HUGO_VERSION}_Linux-64bit.tar.gz" && \
    tar xzf hugo_${HUGO_VERSION}_Linux-64bit.tar.gz && \
    rm -r hugo_${HUGO_VERSION}_Linux-64bit.tar.gz && \
    mv hugo /usr/bin

COPY ./${STATIC_PAGE} /site
WORKDIR /site
RUN hugo
```
###### Build image and override multiple arguments (ARG)
```sh
docker build --build-arg STATIC_PAGE="netgtz.gitlab.io" --build-arg HUGO_VERSION="0.83.1" -t hugo .
```

#### Output
```sh
:~/lab$ docker build --build-arg STATIC_PAGE="netgtz.gitlab.io" --build-arg HUGO_VERSION="0.83.1" -t hugo .
Sending build context to Docker daemon  8.536MB
Step 1/8 : FROM nginx:alpine as build
alpine: Pulling from library/nginx
540db60ca938: Pull complete
0ae30075c5da: Pull complete
9da81141e74e: Pull complete
b2e41dd2ded0: Pull complete
7f40e809fb2d: Pull complete
758848c48411: Pull complete
Digest: sha256:0f8595aa040ec107821e0409a1dd3f7a5e989501d5c8d5b5ca1f955f33ac81a0
Status: Downloaded newer image for nginx:alpine
 ---> a6eb2a334a9f
Step 2/8 : RUN apk add --update     wget
 ---> Running in 96e7303147c7
fetch https://dl-cdn.alpinelinux.org/alpine/v3.13/main/x86_64/APKINDEX.tar.gz
fetch https://dl-cdn.alpinelinux.org/alpine/v3.13/community/x86_64/APKINDEX.tar.gz
(1/3) Installing libunistring (0.9.10-r0)
(2/3) Installing libidn2 (2.3.0-r0)
(3/3) Installing wget (1.21.1-r1)
Executing busybox-1.32.1-r6.trigger
OK: 27 MiB in 45 packages
Removing intermediate container 96e7303147c7
 ---> 8c9b6b201585
Step 3/8 : ARG HUGO_VERSION="0.72.0"
 ---> Running in a4c26ace44df
Removing intermediate container a4c26ace44df
 ---> 3ca9889ae589
Step 4/8 : ARG STATIC_PAGE=""
 ---> Running in b062a028b5b4
Removing intermediate container b062a028b5b4
 ---> 44b5e545b113
Step 5/8 : RUN wget --quiet "https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_${HUGO_VERSION}_Linux-64bit.tar.gz" &&     tar xzf hugo_${HUGO_VERSION}_Linux-64bit.tar.gz &&     rm -r hugo_${HUGO_VERSION}_Linux-64bit.tar.gz &&     mv hugo /usr/bin
 ---> Running in f172e6a54f49
Removing intermediate container f172e6a54f49
 ---> 08ffd9933d8e
Step 6/8 : COPY ./${STATIC_PAGE} /site
 ---> b99bdbfef0e1
Step 7/8 : WORKDIR /site
 ---> Running in 55bbd1daf43d
Removing intermediate container 55bbd1daf43d
 ---> cb83bc651786
Step 8/8 : RUN hugo
 ---> Running in 19c79b409639
Start building sites …

                   | EN
-------------------+-----
  Pages            | 32
  Paginator pages  |  0
  Non-page files   |  0
  Static files     | 13
  Processed images |  0
  Aliases          |  0
  Sitemaps         |  1
  Cleaned          |  0

Total in 372 ms
Removing intermediate container 19c79b409639
 ---> 772bcf6d1bf9
Successfully built 772bcf6d1bf9
Successfully tagged hugo:latest
```

#### Current images
```sh
~/lab$ docker images
REPOSITORY   TAG       IMAGE ID       CREATED              SIZE
hugo         latest    772bcf6d1bf9   About a minute ago   81.4MB
nginx        alpine    a6eb2a334a9f   10 days ago          22.6MB
```

#### Running a container using the hugo image
* Using the command '/bin/ash' to keep the container running
* Exposing container port 80 on local port 80
* Passing --rm to delete container after stopping it - no need to do 'docker rm [Container ID]'
    ```sh
    docker run --rm -p 80:80 -dit --name hugoapp hugo /bin/ash
    ```
    
* Attaching to running container
    ```sh
    docker exec -it hugoapp /bin/ash
    ```
    
* Rerunning hugo command to generate site and get its Output
    ```sh
    ~/lab$ docker exec -it hugoapp /bin/ash
    /site # ls
    archetypes   assets       config.toml  content      i18n         layouts      public       resources    static       themes
    /site # hugo
    Start building sites …

                       | EN
    -------------------+-----
      Pages            | 32
      Paginator pages  |  0
      Non-page files   |  0
      Static files     | 13
      Processed images |  0
      Aliases          |  0
      Sitemaps         |  1
      Cleaned          |  0

    Total in 362 ms
    ```

* Moving hugo public folder content to nginx working directory
    ```sh
    mv public/* /usr/share/nginx/html/
    ```
    
* Running the nginx server
    ```sh
    /usr/sbin/nginx
    ```

* Nginx Output

    ```sh
    /site # /usr/sbin/nginx
    2021/06/04 22:01:54 [notice] 18#18: using the "epoll" event method
    2021/06/04 22:01:54 [notice] 18#18: nginx/1.21.0
    2021/06/04 22:01:54 [notice] 18#18: built by gcc 10.2.1 20201203 (Alpine 10.2.1_pre1)
    2021/06/04 22:01:54 [notice] 18#18: OS: Linux 4.15.0-20-generic
    2021/06/04 22:01:54 [notice] 18#18: getrlimit(RLIMIT_NOFILE): 1048576:1048576
    /site # 2021/06/04 22:01:54 [notice] 19#19: start worker processes
    2021/06/04 22:01:54 [notice] 19#19: start worker process 20
    ```
