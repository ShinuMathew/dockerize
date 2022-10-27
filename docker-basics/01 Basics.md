# Docker Overview

`Docker is platform that provides a system for development, deployment of containerized application. Basically it helps us to achieve containerization`

## Containers :
* Containers are completely isolated environments. They can have their own processes, services, network interfaces, their own mounts just like VMs except that they `share the same OS Kernel`
![Containers](assets/Containers.png "Containers")
* Containers are not new. They have existed before Docker. There are different types LXC, LXD, LXCFS, etc.
* Docker utilizes LXC containers. Setting up these containers are very hard since it is very low level.
* That is where docker comes in as a high level tool and provides various powerful functionalities and tools, making it easier for the users like us.

## How docker works
* The linux OS distros like Ubuntu, CetntOS, Fedora, etc consists of OS kernel and a set of software.
* Kernel interacts with hardware. Its same for all linux destros. 
* The software above it makes these OS different. They have different UI, Driver, Compilers, File managers, dev tools, etc. That is a common linus Kernel shared across OS's and custom softwares which differs.
![Kernel](assets/Kernel.png "Kernel")
* As discussed above, the docker containers share the same OS kernel. What does that mean?
* Let's say we have an ubuntu system with docker installed on it. Docker can run any flavour of OS on top of it since kernel is same - Linux.
* Docker utilizes the underlying kernel of the docker host which works for other distros.
* Now this means docker wont support directly on Windows.
* Docker for windows does not actually runs a linux container directly on windows. It runs linux container on a linux VM on Windows.

## Container vs VM
* VM requires Hypervisor like ESXi to run multiple OS.
* Container has less isolation since the OS is same but VM has more isolation.
![Container-vs-VM](assets/Container-vs-VM.png "Container-vs-VM")
* Containers like docker we have `Hardware infrastructure`, then the `OS` and then `Docker installed on the OS`. Docker then manages the containers that run with `libraries and dependencies`.
* Virtual machines, we have a `Hypervisor` over the `Hardware infrastructure` and then teh VM's on them.
* Each `VM has its own OS` and then the `libraries and dependencies`.
* The overheads are that, 
    * Higher utilization of underlying resources in VM as there are multiple OS and kernels running.
    * VMs take higher disk size and are in GB. Each Vm is heavy whereas docker containers are light weight in MB.
    * This causes the containers to boot up immediately in seconds whereas VMs take minutes.
    * Docker hence also has less isolation since the kernels and resources are shared unlike VMs
* However its `not containerization or virtualization` in docker its `containers & virtual machines` 
* When we have large environment with 1000s of docker containers running on 1000s of docker hosts, we often see containers provisioned on `virtual docker hosts`.
* This way we can use the advantage of both technologies. We can use virtulaization to easily provision or decommision docker hosts as required. At the same time use benefits of Docker to easily provision application and quickly scale them when required.
* However in this case we wont be provisioning too many VMs as before because earlier we provisioned a VM for each app. Now we do it for 100s or 1000s of containers.
![Container-and-VM](assets/Container-and-VM.png "Container-and-VM")

* Most organisation have their images in docker hub.
* We can just pull these image and run it to up the container with the required instance.
![docker-images](assets/docker-images.png "Docker-images")
* We can run multiple instances of VM by using docker run command for same image and configure a load balancer. If one instance fails we can remove it and create a new one with docker run

## Docker images
* An image is a package or template like VM templates used to create one or more images.
* Containers are running instances of the images that are isolated and have their own environment and set of processes.
![DockerImage](assets/DockerImage.png "DockerImage")

* Traditionally developers develop the app and package like a .war and share and OPS team will setup the environment based on dev instructions.
* This causes confusion and issues for the operations/devops team to setup the environment.
![DockerDevops](assets/DockerDevops.png "DockerDevops")
* With docker, developer and ops team work hand in hand to create a dockerfile. Dockerfile creates image which is guranteed to run in the environment.
![DockerDevops2](assets/DockerDevops2.png "DockerDevops2")
![DockerDevops3](assets/DockerDevops3.png "DockerDevops3")

## Dockerfile

* Dockerfile is used to create an image of the docker container
```dockerfile
    FROM node
    # Pulls an image of latest node
    WORKDIR /usr/src/app
    # Creates a work directory /usr/src/app
    COPY package*.json ./
    # Copies the package.json file
    RUN npm install
    # install dependencies
    COPY . .
    # Copies all the content from the project folder to the workdir
    EXPOSE 5001
    # Exposes port 5001
    CMD ["npm", "start"]
    # Runs npm start on docker run
```
* To build docker image : `docker build -t node-auth/demo .`
* To create and run the container : `docker run -p 5001:5001 node-auth/demo`
* To check the running docker containers : `docker ps`
```bash
CONTAINER ID   IMAGE            COMMAND                  CREATED              STATUS              PORTS                    NAMES
5d7c64b9bd2b   node-auth/demo   "docker-entrypoint.s…"   About a minute ago   Up About a minute   0.0.0.0:5001->5001/tcp   loving_hodgkin
```
* To stop the process : ` docker stop 5d7c64b9bd2b`. This will stop the process gracefully.
* To kill a docker process : `docker kill 3d563c621f26`. This will kill the process.

## Docker Compose

* Dockerfile is a simple text file that contains commands a user would call to assemble an image. 
* A docker compose is a tool for defining and running multi-container docker applications.
* A simple docker compose file for running a node application with mongodb would be like below,
```yaml
services:
  app:
    container_name: app
    restart: always
    build: .
    ports:
      - "3000:3000"
    links:
      - mongo
  mongo:
    container_name: mongo
    image: mongo
    volumes:
      - ./data:/data/db
    ports:
      - "27017:27017"
```
* This simple docker compose creates 2 containers based on the service, `app` & `mongo`.
* The service mongo here pulls an existing image from docker hub.
* It requires to store the data in docker volume. Hence we are specifying a `volume` here.
* The app service however is build using the dockerfile.
* Once the compose file is created, to build the container image run `docker-compose build`
* To run each service, run : `docker-compose up -d mongo`
* Since our service `app` is dependent on mongo db we first run `mongo` container and then the `app`
* Inside the code, our mongodb connection url should point to the docker container `mongo` instead of local host
```javascript
    db: {
        protocol: 'mongodb',
        host: 'mongo',  // Docker mongo container
        port: "27017:27017",
    }
```
* Run `docker ps` and it will list the running containers.
* To check the logs, we can run : `docker logs e136ee195b12`