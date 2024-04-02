# The Docker Tutorial

I found this interesting tutorial on Laracasts. It's by Andrew Schmelyun.

### Episode 1 - What is Docker

Docker is a software platform that uses containers to build, share and run applications.<br>
You can use [Docker Desktop](https://www.docker.com/products/docker-desktop/).<br>
Alternatively, you can follow these steps [Install Docker Engine on Ubuntu](https://docs.docker.com/engine/install/ubuntu/) and [Install Docker Compose](https://docs.docker.com/compose/).<br>

Now Docker is installed. How can we use it?<br>
First we open a terminal and enter the following command:
```
docker run hello-world
```
The command pull down the ["hello-world"](https://hub.docker.com/_/hello-world) container from the docker hub.<br>
To remove the container completely just use the following command:
```
docker run --rm hello-world
```
or delete it using the delete icon in the Docker desktop application.

To get version information about the pulled container just use the following command:
```
docker run --rm ubuntu:latest cat /etc/os-release
```

### Episode 2 - Create a Simple Docker File

First we create a custom Dockerfile:
```
touch Dockerfile
code Dockerfile
```
The aim of this file is to provide us a set of instructions for docker to use that builds and prepares our container when we use docker run.<br>
Lets think of something we can do with this container. How about to create a container for a [JSON Server](https://www.npmjs.com/package/json-server).<br>

The first thing that has to be included in every Dockerfile is the "FROM" command:
```
FROM node:latest
```
This is followed by the base image that you want to build your container.<br>

Next we use the "WORKDIR" command to determine in what directory we are going to be in when we run any following commands:
```
WORKDIR /home/server
```
If the directory isnt available it will be created.<br>

To execute instructions like installing a npm package we can use the "RUN" command:
```
RUN npm install -g json-server
```
For the JSON Server we need a file for its data:
```
touch db.json
code db.json
```
with the following example data:
```
{
  "posts": [
    {
      "title": "My first Post",
      "author": "web devapps"
    },
    {
      "title": "My second Post",
      "author": "web devapps"
    }
  ],
  "profile": {
    "first_name": "Web",
    "last_name": "Devapps",
    "email": "ich@webdevapps.de"
  }
}
```
In order to be able to use the data file in our Docker container we can use the "COPY" command to copy the file into the container:
```
COPY db.json /home/server/db.json
```

The last thing we need to do is tell Docker to actually run json-server. There are two ways to do it:
1. "ENTRYPOINT"
2. "CMD"

"ENTRYPOINT" is the main command ran by the Docker container. It can not be overwritten by commands on the host machine instead commands are added to it.<br>

"CMD" is for additional commands or options that will run after the "ENTRYPOINT" command.<br>

For now we use the "ENTRYPOINT" command:
```
ENTRYPOINT ["json-server", "db.json"]
```
There is also the alternative shell form:
```
ENTRYPOINT /bin/sh -c json-server db.json
```
But we use the exact form as array notation above.
