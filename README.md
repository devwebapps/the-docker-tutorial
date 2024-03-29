# the-docker-tutorial

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
