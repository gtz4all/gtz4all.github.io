---
title: "HTTP Server with Dynamic Content"
date: 2021-06-12T15:06:47Z
draft: false
description: "HTTP Server with Dynamic Content using scripts written in common programming languages"
---


#### Apache HTTPD CGI

Apache is an open-source HTTP server (httpd) that has been around since 1995. The HTTPD CGI (Common Gateway Interface) module provides a method to display dynamic content on web sites using scripts written in any language such as python, perl or bash. https://httpd.apache.org/

* Python CGI module
The CGI Module can be use to evaluate and process user input submitted through an HTML <form> or javascript function() using the FieldStorage class.
The FieldStorage supports indexed Python dictionary, standard dictionary method keys() and the built-in function len(). Form fields containing empty strings are ignored and do not appear in the dictionary; to keep such values, provide a true value for the optional keep_blank_values keyword parameter when creating the FieldStorage instance.

```python
    # create key/value pairs form fields passed by javascript/html
    form = cgi.FieldStorage()
    form_vars = {}
    for key in form.keys():
       form_vars[key] = form[key].value
```


#### Building an Apache-CGI Docker Image using a Dockerfile

Docker is virtualization tool used to create lightweight, portable images and containers. Docker images are self-contained, meaning they dont have any dependencies on the uderlaying OS. As long as you have docker installed, you'll be able to build a container with any docker image.

###### Instructions

There are many prebuild httpd-cgi docker images out there(Docker Hub). However I took a few for a test drive and noticed they didnt have what I needed. In order to address some of these limitations, I had two options: Build an image on top of those prebuilt images or build one from scratch. I decided to build one from scratch in order to fully customize it to my needs. Some of the packages are for future use such as requests and boto3.


###### Alpine Base image
Alpine Linux is a 3MB lightweight Linux Image with a large selection of packages it is package manager apk. https://alpinelinux.org/about/

###### Software requirements
These software requirements are specific to my webform needs. I use Python3 CGI to collect submitted form key,values, Jinja2 to generate dynamic HTML pages, request to make Rest API calls, and boto3 to directly make use of AWS resources. Nano is a plain and simple text editor.

Software|Description
--|--
apache2|HTTP Server
python3|CGI Script prefered programing language
pip3|Used to install other python modules
jinja2|Used to HTML render templates
request|used to make HTTP GET/POST calls to Rest APIs
boto3|Used to interact with AWS Resources
nano|Simple text editor

###### Enable CGI by Modifying HTTPD Config - httpd.config
Since I dont spend too much time doing web development work. I try to avoid making extensive changes to the default httpd.config file in order to avoid unexpected behavior. 

```bash
## omitted some not so relevant items
 ##** Enabled CGI Module **##
LoadModule cgi_module modules/mod_cgi.so
##** send logs to stdout - docker logs **##
CustomLog /proc/self/fd/1 format
##** Avoids attempting to invoke directoy as script **##
Alias "/" "/var/www/localhost/cgi-bin/"
##** only runs cgi,pl,py as scripts **##
AddHandler cgi-script cgi pl py sh
Options ExecCGI
```

###### Dockerfile
A Dockerfile is a text document that contains all the commands to assemble an image. The docker build command executes the command-line instructions in the Dockerfile one-by-one, committing the result of each instruction to a new image(if necessary) before finally outputting the new image ID. 

> If necessary, the image can by modified by editing the Dockerfile and rerunning the docker build command using the same tag provided during the initial built.

```Dockerfile
###--------------------START-------------------------------------#
################################################################
# httpd:Alpine
# Features: CGI, Python3 - Jinja2, Requests, Boto3 
################################################################

# Base Image - Official Alpine
FROM alpine:latest

LABEL vendor="Gtz4All"
LABEL maintainer="netgtz"

# Upgrade existing packages in the base image
RUN apk --no-cache upgrade

# Install apache from packages with out caching install files
RUN apk add --update --no-cache nano apache2 python3 && ln -sf python3 /usr/bin/python
RUN python3 -m ensurepip
RUN pip3 install --no-cache --upgrade pip setuptools jinja2 requests boto3

# backup httpd.conf
RUN mv /etc/apache2/httpd.conf /etc/apache2/httpd.conf.bk

# Copy custom files
COPY httpd.conf /etc/apache2/httpd.conf
COPY cgi-bin/ /var/www/localhost/cgi-bin

# Open port for httpd access
EXPOSE 80

# Run httpd FOREGROUND in the background(-D option)
CMD ["-D","FOREGROUND"]

# Start httpd
ENTRYPOINT ["/usr/sbin/httpd"]

###--------------------END-------------------------------------#
```

###### Building the docker image 


> Image Tags - you can create multiple tags for the same docker image, they will show up in 'docker images'. Since they are just that, tags, they are not consuming any additional disk space. 


Options|Description
--|--
-t|image tag name
-f|custom Dockerfile name
.| Default Dockerfile name  

```bash
docker build -t custom-httpd-cgi .
```

###### Docker Image Built Summary
The Custom Dockerfile pulled the latest alpine image and installed apache2, python3, pip3, jinja2, requests, boto3 and nano. It renamed the original httpd.config file and copied the custom version. **Final image size - 150MB**.  
    
#### Running The Docker Container
These are some of the commonly options used when working with containers.
Options|Description
--|--
-d|deattach
-i|interactive
-t|terminal
-p|portmapping [local-port]:[container-port]
--name|custom name
--rm|Clean up (--rm) removes container after stopping it

###### run container in deattach mode
This command runs the container in deattached mode, mapping local port 8080 to container port 80

```bash
docker run -d --name webserver -p 8080:80 custom-httpd-cgi
```
* **Verify image is running**

```bash
$ docker ps
CONTAINER ID   IMAGE              COMMAND                  CREATED              STATUS              PORTS                                   NAMES
4ca72abac592   custom-httpd-cgi   "/usr/sbin/httpd -D …"   About a minute ago   Up About a minute   0.0.0.0:8080->80/tcp, :::8080->80/tcp   webserver
```
* **Verify webserver is running**

> Output came from /cgi-bin/index.html

```bash
$ curl localhost:8080
<!DOCTYPE html>
<html>

<head>
  <title>Gtz4All</title>
</head>

<body>
  <h1><center>Welcome to Gtz4All Demo Page</center></h1>
  <p><center>To learn more, please visit <a href="https://netgtz.gitlab.io/">Gtz4All Blog</center></a></p>
</body>

</html>
```

* **Environment Variables**

```bash
$ curl localhost:8080/show-env.py

CONTEXT_DOCUMENT_ROOT = /var/www/localhost/cgi-bin/</br/>
CONTEXT_PREFIX = /</br/>
DOCUMENT_ROOT = /var/www/localhost/htdocs</br/>
GATEWAY_INTERFACE = CGI/1.1</br/>
HTTP_ACCEPT = */*</br/>
HTTP_HOST = localhost:8080</br/>
HTTP_USER_AGENT = curl/7.58.0</br/>
PATH = /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin</br/>
QUERY_STRING = </br/>
REMOTE_ADDR = 172.17.0.1</br/>
REMOTE_PORT = 49684</br/>
REQUEST_METHOD = GET</br/>
REQUEST_SCHEME = http</br/>
REQUEST_URI = /show-env.py</br/>
SCRIPT_FILENAME = /var/www/localhost/cgi-bin/show-env.py</br/>
SCRIPT_NAME = /show-env.py</br/>
SERVER_ADDR = 172.17.0.2</br/>
SERVER_ADMIN = you@example.com</br/>
SERVER_NAME = localhost</br/>
SERVER_PORT = 8080</br/>
SERVER_PROTOCOL = HTTP/1.1</br/>
SERVER_SIGNATURE = <address>Apache/2.4.48 (Unix) Server at localhost Port 8080</address>
</br/>
SERVER_SOFTWARE = Apache/2.4.48 (Unix)</br/>
```
![show-env.py](images/show-env-py.JPG)


* **Attach to Container terminal**

    Every image has its own shell terminal - ubuntu = /bin/bash, Alpine = /bin/sh

```
docker exec -it webserver /bin/sh
```

#### Mounting webservices folder

This folder contains a sample webform used as a base to provision/build/deploy resources. It makes use of Javascript,Jinja templates, and CGI.

> Mounting folder keeps all files changes local [local-folder]:[container-folder] [container-name]


###### map local folder to container's cgi-bin folder
```
docker run -d -p 8080:80 -v $(pwd)/webservices:/var/www/localhost/cgi-bin/webservices --name webserver custom-httpd-cgi
```
![webrequest.cgi](images/webrequest-cgi.JPG)



#### registry.gitlab.com
###### Pushing image to registry.gitlab.com

* **Login to Docker registry**
```
docker login registry.gitlab.com --username netgtz
```
* **Current Image - Rename to Standard**
```
docker rename custom-httpd-cgi registry.gitlab.com/netgtz/httpd-cgi/custom-httpd-cgi
```
* **Build Docker Image**
```
docker build -t registry.gitlab.com/netgtz/httpd-cgi/custom-httpd-cgi .
```
* **Push Docker Image**
```
docker push registry.gitlab.com/netgtz/httpd-cgi/custom-httpd-cgi
```


###### Pull image from registry.gitlab.com
* **Login to Docker registry**
```
docker login registry.gitlab.com --username netgtz
```
* **Pull Image**
```
docker pull registry.gitlab.com/netgtz/httpd-cgi/custom-httpd-cgi
```


