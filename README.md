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

The first thing that has to be included in every Dockerfile is the `FROM` command:
```
FROM node:latest
```
This is followed by the base image that you want to build your container.<br>

Next we use the `WORKDIR` command to determine in what directory we are going to be in when we run any following commands:
```
WORKDIR /home/server
```
If the directory isnt available it will be created.<br>

To execute instructions like installing a npm package we can use the `RUN` command:
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
In order to be able to use the data file in our Docker container we can use the `COPY` command to copy the file into the container:
```
COPY db.json /home/server/db.json
```

The last thing we need to do is tell Docker to actually run json-server. There are two ways to do it:
1. `ENTRYPOINT`
2. `CMD`

`ENTRYPOINT` is the main command ran by the Docker container. It can not be overwritten by commands on the host machine instead commands are added to it.<br>

`CMD` is for additional commands or options that will run after the `ENTRYPOINT` command.<br>

For now we use the "ENTRYPOINT" command (array notation):
```
ENTRYPOINT ["json-server", "db.json"]
```
There is also the alternative shell form:
```
ENTRYPOINT /bin/sh -c json-server db.json
```
But we use the exact form as array notation above.<br>

Now its time to build the container:
```
docker build .
```
After the Docker container has been created, we can display the docker images with:
```
docker image list
```
| REPOSITORY | TAG      | IMAGE ID     | CREATED            | SIZE    |
|------------|----------| ------------ | ------------------ |---------|
| `<none>`   | `<none>` | 0c3f3aa2ecc3 | About a minute ago | 1.13GB  |

We copy the Image ID and put it to use
```
docker run --rm 0c3f3aa2ecc3
```
Now we see the "Hello Screen" from json server. Lets see if we can load the domain:
```
http://localhost:3000
```
What a surprise we get an error page. What's the problem?
The json server is running on port 3000 but that port is isolated inside of the docker container.
Its not exposed to our local machine and so we can't send or receive any data from it.
We have to change that.

First let's bring down the docker container thats running. In a new terminal tab run:
```
docker ps
```
| CONTAINER ID | IMAGE        | COMMAND               | CREATED        | STATUS        | PORTS | NAMES             |
| ------------ | ------------ | --------------------- | -------------- | ------------- |-------|-------------------|
| 1349d3d6440d | 0c3f3aa2ecc3 | "json-server db.json" | 12 minutes ago | Up 12 minutes |       | trusting_franklin |
This command shows all currently running images. We copy the id of our container and run:
```
docker stop 1349d3d6440d
```
Let's open up our Dockerfile again:
```
FROM node:latest

WORKDIR /home/server

RUN npm install -g json-server

COPY db.json /home/server/db.json

ENTRYPOINT ["json-server", "db.json"]
```
After our `COPY` Statement we add another command called `EXPOSE`. This expects a port number 
and gives our container the ability to map a port from our host machine into the docker container
at the port specifed. By default we saw that json server runs on port 3000:
```
EXPOSE 3000
```
By default json server runs on localhost but we need to specify a different ip address.
In our ENTRYPOINT lets add the --host flag and the address 0.0.0.0:
```
ENTRYPOINT ["json-server", "db.json", "--host", "0.0.0.0"]
```
The ip address `0.0.0.0` is the default docker bridge network ip essentially connecting our host machine
to the docker instance. Our dockerfile now looks like this:
```
FROM node:latest

WORKDIR /home/server

RUN npm install -g json-server

COPY db.json /home/server/db.json

EXPOSE 3000

ENTRYPOINT ["json-server", "db.json", "--host", "0.0.0.0"]
```
Lets rebuild the container:
```
docker build . --no-cache
```
The `--no-cache` flag disregard any cache files that let have changed. Copy the image id and run:
```
docker run --rm -p 3000:3000 2f269b32d46d
```
With the `-p` flag we can choose which port on our local machine we bind that to.
`-p 3000:3000` means that the first number is the local system and the second number is the docker container.
Lets have a look in our browser what happens if we call `http://localhost:3000/`. Fine we see the json server
landing page as expected and see the data we provided.

#### How we can use the CMD to specify some optional arguments:
Lets try to use the `CMD Command` with some optional arguments. Bring down the actual running image container. 
Open the Dockerfile and add under the `COPY`command another `COPY` command and remove from the `ENTRYPOINT` command the second argument `db.json`:
```
...

COPY db.json /home/server/db.json

COPY old.json /home/server/old.json

ENTRYPOINT ["json-server", "--host", "0.0.0.0"]

CMD ["db.json"]
```
In our project directory we copy the `db.json` as `old.json`
```
cp db.json old.json
```
Lets make a few changes to old.json so that we know there is a difference:
```
{
  "posts": [
    {
      "title": "My alternative first Post",
      "author": "web devapps two"
    },
    {
      "title": "My alternative second Post",
      "author": "web devapps two"
    }
  ],
  "profile": {
    "first_name": "Web",
    "last_name": "Devapps Two",
    "email": "ich2@webdevapps.de"
  }
}
```
Lets rebuild our docker container. With the `-t json-server` flag we can specify an image name:
```
docker build . --no-cache -t json-server
```
Now we can run with and see the expected landing page with the posts of `db.json`:
```
docker run --rm -p 3000:3000 json-server
```
If we bring the container down and rerun it with the specifying the `old.json` file at the end:
```
docker run --rm -p 3000:3000 json-server old.json
```
Now we see the landing page with the alternative posts from `old.json`. Nice :-)