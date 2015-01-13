# OpenShift v3

## An evolution that will change everything

---

<!-- .slide: data-background="../images/docker/docker-wallpaper-grey.jpg" -->

--

# What is Docker

* Designed to deliver applications faster
* Separates Application from Infrastructure
* Provides Tools to put Applications into Containers
* Image Registry to host and distribute images
  * Locally or in the Cloud

Notes: Docker is an open platform for developing, shipping, and running applications. Docker is designed to deliver your applications faster. With Docker you can separate your applications from your infrastructure AND treat your infrastructure like a managed application. Docker helps you ship code faster, test faster, deploy faster, and shorten the cycle between writing code and running code.
Docker does this by combining a lightweight container virtualization platform with workflows and tooling that help you manage and deploy your applications.
At its core, Docker provides a way to run almost any application securely isolated in a container. The isolation and security allow you to run many containers simultaneously on your host. The lightweight nature of containers, which run without the extra load of a hypervisor, means you can get more out of your hardware.
Surrounding the container virtualization are tooling and a platform which can help you in several ways:
getting your applications (and supporting components) into Docker containers
distributing and shipping those containers to your teams for further development and testing
deploying those applications to your production environment, whether it be in a local data center or the Cloud.

--

# What is Dockers' Use Case

* Faster delivery of your applications
* Deploying and scaling more easily
* Achieving higher density and running more workloads

Notes: Docker is perfect for helping you with the development lifecycle. Docker allows your developers to develop on local containers that contain your applications and services. It can then integrate into a continuous integration and deployment workflow.
For example, your developers write code locally and share their development stack via Docker with their colleagues. When they are ready, they push their code and the stack they are developing onto a test environment and execute any required tests. From the testing environment, you can then push the Docker images into production and deploy your code.
Docker's container-based platform allows for highly portable workloads. Docker containers can run on a developer's local host, on physical or virtual machines in a data center, or in the Cloud.
Docker's portability and lightweight nature also make dynamically managing workloads easy. You can use Docker to quickly scale up or tear down applications and services. Docker's speed means that scaling can be near real time.
Docker is lightweight and fast. It provides a viable, cost-effective alternative to hypervisor-based virtual machines. This is especially useful in high density environments: for example, building your own Cloud or Platform-as-a-Service. But it is also useful for small and medium deployments where you want to get more out of the resources you have.

--

# Major Components

## Docker has 2 major components

* Docker: the open source container virtualization platform
* [Docker Hub](https://hub.docker.com/): our Software-as-a-Service platform for sharing and managing Docker containers.

--

## Docker Architecture

![Docker Architecture](../images/docker/architecture.svg) <!-- .element: class="stretch" -->

Notes:
The Docker daemon
As shown in the diagram above, the Docker daemon runs on a host machine. The user does not directly interact with the daemon, but instead through the Docker client.
The Docker client
The Docker client, in the form of the docker binary, is the primary user interface to Docker. It accepts commands from the user and communicates back and forth with a Docker daemon.

--

# Inside Docker

#### To understand Docker's internals, you need to know about three components

- Docker images
- Docker registries
- Docker containers

Notes:
Docker images
A Docker image is a read-only template. For example, an image could contain an Ubuntu operating system with Apache and your web application installed. Images are used to create Docker containers. Docker provides a simple way to build new images or update existing images, or you can download Docker images that other people have already created. Docker images are the build component of Docker.
Docker Registries
Docker registries hold images. These are public or private stores from which you upload or download images. The public Docker registry is called Docker Hub. It provides a huge collection of existing images for your use. These can be images you create yourself or you can use images that others have previously created. Docker registries are the distribution component of Docker.
Docker containers
Docker containers are similar to a directory. A Docker container holds everything that is needed for an application to run. Each container is created from a Docker image. Docker containers can be run, started, stopped, moved, and deleted. Each container is an isolated and secure application platform. Docker containers are the run component of Docker.

--

# Putting it all to work

#### So far we have learned that

1. You can build Docker images that hold your applications
2. You can create Docker containers from those Docker images to run your applications
3. You can share those Docker images via [Docker Hub](https://hub.docker.com/) or your own registry

#### Let's look at how these elements combine together to make Docker work. <!-- .element: class="fragment" data-fragment-index="1" -->

--

# Docker Image

- Docker Images are built from base images
- Images are extended using a simple and descriptive set of instructions
  - Run a command
  - Add a File or Directory
  - Create an Environment Variable
  - What process to run when launching a container from this image

These Instructions are stored in a file called `Dockerfile`. Docker reads this `Dockerfile` when you request a build of an image, executes the instructions, and returns a final image. <!-- .element: style="padding-top: 25px" -->

Notes:
How does a Docker Image work?
We've already seen that Docker images are read-only templates from which Docker containers are launched. Each image consists of a series of layers. Docker makes use of union file systems to combine these layers into a single image. Union file systems allow files and directories of separate file systems, known as branches, to be transparently overlaid, forming a single coherent file system.
One of the reasons Docker is so lightweight is because of these layers. When you change a Docker image—for example, update an application to a new version— a new layer gets built. Thus, rather than replacing the whole image or entirely rebuilding, as you may do with a virtual machine, only that layer is added or updated. Now you don't need to distribute a whole new image, just the update, making distributing Docker images faster and simpler.
Every image starts from a base image, for example RHEL6, a base RHEL6 image, or fedora, a base Fedora image. You can also use images of your own as the basis for a new image, for example if you have a base Apache image you could use this as the base of all your web application images.
These Images are ususally pulled or downloaded from dockerhub.

--

# Docker Registry

The Docker registry is the store for your Docker images. Once you build a Docker image you can push it to a public registry i.e. [Docker Hub](https://hub.docker.com/) or to your own registry running behind your firewall.

```
$ sudo docker push yourname/newimage
```
--

Using the Docker client, you can search for already published images

```
$ sudo docker search centos
NAME           DESCRIPTION                                     STARS     OFFICIAL   TRUSTED
centos         Official CentOS 6 Image as of 12 April 2014     88
tianon/centos  CentOS 5 and 6, created using rinse instea...   21
...
```

and then pull them down to your Docker host to build containers from them.

```
$ sudo docker pull centos
Pulling repository centos
0b443ba03958: Download complete
539c0211cd76: Download complete
511136ea3c5a: Download complete
7064731afe90: Download complete

Status: Downloaded newer image for centos
```

and build containers from them...

```
$ sudo docker build -t my_custom_image .
```
