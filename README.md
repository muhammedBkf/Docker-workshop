# Docker workshop
## Installation

1. First we check if we have docker already installed:
```bash
sudo docker -v
```
If nothing appeared You can try it online : https://labs.play-with-docker.com/

Or we proceed to the installation.

2. **Installing Docker on your Linux distribution**
- On Fedora
```bash
sudo yum install docker-io
```
- On CentOS 7: 
```bash
sudo yum install docker
``` 
- On Debian and derivatives: 
```bash
sudo apt-get install docker.io
```

- You can use the curl command to install on several platforms: 
```bash
curl -s https://get.docker.com/ | sudo sh
```

3. **Running without `sudo`:**
- The docker user is root equivalent. 
- It provides root-level access to the host. 
- You should restrict access to it like you would protect root.
- If you give somebody the ability to access the Docker API, you are giving them full access on the machine.
### **The docker group**
- Add the Docker group 
```bash
sudo groupadd docker 
```
- Add ourselves to the group 
```bash
sudo gpasswd -a $USER docker
```
- Restart the Docker daemon 
```bash
sudo service docker restart 
```
- Log out

- Check that Docker works without sudo
```bash
docker -v
```


## Docker architecture
Docker is a client-server application.
- **The Docker daemon (or "Engine")**
	Receives and processes incoming Docker API requests.

- **The Docker client**
	Talks to the Docker daemon via the Docker API. We'll use mostly the CLI embedded within the docker binary.

- **Docker Hub Registry**
	Collection of public images. The Docker daemon talks to it via the registry API.

### Your first container 
```bash
docker run alpine echo hello world
```
- We used one of the smallest, simplest images available: alpine. 
- alpine is typically used in dockerizing applications. 
- We ran a single process and echo'ed hello world.

### A more useful container
```bash
docker run -it ubuntu bash
```
- This is a brand new container.
- -it is shorthand for -i -t.
	- -i tells Docker to connect us to the container's stdin.  
	- -t tells Docker that we want a pseudo-terminal.
	
We do something in our container

```bash
figlet "Hi Nexusers"
```
We must install it first

```
apt update
apt install figlet
```

We try again:
```bash
figlet "Hi Nexusers"
```
then we exit.

- Starting another container
```bash
docker run -it ubuntu bash
```
And we try to run the same command 
```
figlet
```

We will see in the next chapters how to bake a custom image with figlet.
## Background Containers
Our first containers were interactive.
### A non-interactive container
```bash
docker run jpetazzo/clock
```
- This container will run forever. 
- To stop it, press ^C. 
- Docker has automatically downloaded the image jpetazzo/clock. 
- This image is a user image, created by jpetazzo. 
- We will tell more about user images (and other types of images) later
```bash
docker run -d jpetazzo/clock
```


### List running containers

How can we check that our container is still running?
```bash
docker ps
```

To see only the last container that was started:
```bash
docker ps -l
```
To see only IDs
```bash
docker ps -q
```
U can combine them
```bash
docker ps -lq
```
### View the logs of a container
```bash
docker logs docker_id
```

Follow the logs in real time

```
docker logs --follow docker_id
```


### Stop our container
We can:
- Killing it using the docker kill command.
- Stopping it using the docker stop command.

### Background and foreground
- The distinction between foreground and background containers is arbitrary. 
- From Docker's point of view, all containers are the same. 
- All containers run the same way, whether there is a client attached to them or not. 
- It is always possible to detach from a container, and to reattach to a container


## Understanding Docker Images

- An image is a collection of files + some meta data. (Technically: those files form the root filesystem of a container.) 
- Images are made of layers, conceptually stacked on top of each other. 
- Each layer can add, change, and remove files.

### Differences between containers and images

#### Object-oriented programming 
- Images are conceptually similar to classes. 
- Layers are conceptually similar to inheritance. 
- Containers are conceptually similar to instances

### A chicken-and-egg problem 
- The only way to create an image is by "freezing" a container. 
- The only way to create a container is by instantiating an image. 
- Help!

	There is a special empty image called **scratch**. • It allows to build from scratch.

### Wait a minute...
If an image is read-only, how do we change it?

- We don't. 
- We create a new container from that image. 
- Then we make changes to that container. 
- When we are satisfied with those changes, we transform them into a new layer.
- A new image is created by stacking the new layer on top of the old image


To create a new image we use :
```bash
docker commit
```
- Saves all the changes made to a container into a new layer.
- Creates a new image (effectively a copy of the container).

We make the changes we want then:
```
docker commit docker_id  Nexus/docker_workshop:version1
```

### Dockerfile

- Manual process = bad. 
- Automated process = good
We will build a container image automatically, with a Dockerfile.


- A Dockerfile is a build recipe for a Docker image. 
- It contains a series of instructions telling Docker how an image is constructed. 
- The docker build command builds an image from a Dockerfile

Create a Dockerfile and write this in it:
```bash
FROM ubuntu 
RUN apt-get update 
RUN apt-get install -y figlet
```

- FROM indicates the base image for our build. 
- Each RUN line will be executed by Docker during the build. 
- Our RUN commands must be non-interactive. (No input can be provided to Docker during the build.)

Save our file, then execute:
```
docker build -t figlet .
```
