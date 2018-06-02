# Docker Tutorial

Following the official Docker [getting started guide](https://docs.docker.com/get-started/).

## Containers

Docker automates the deployment of software applications inside containers. In other words, the app is packaged with all dependencies into a standardized unit. A container runs natively on linux and shares the kernel of the host with other containers. This provides the isolation of VMs at a fraction of the cost. 

**Image**: the blueprint of the app--an executable packing of everything the app needs to run (code, runtime, libs, conf, etc).

**Container**: An instance of an image when it is run.

**Docker Daemon**: A background service running on the host that manages building, running, and distributing docker containers. The client talks to this.

**Docker Client**: The command line tool used to interact with the daemon. GUI clients also exist.

**Docker Hub**: A registry of hubs. Note: hosting of a private hub is also possible. 

**Dockerfile**: A file defining the configuration of the environment for your container and running the application. 

### Common commands

```
## List Docker CLI commands
docker
docker container --help

## Display Docker version and info
docker version
docker info

## Execute Docker image
docker run <image-name>

## List Docker images
docker image ls

## List Docker containers 
docker container ls
docker container ls --all 

## Create Docker image using current dir
docker build -t <tag_name> .

## Run the app in detached mode with -d 
## use -p to map your machines port 4000 to container's published port 80
docker run -d -p 4000:80 friendlyhello

## Stop a Docker container
docker container stop <container_id>

## Publish a tagged image
docker push username/repository:tag

```

### Example Dockerfile

```
# Use an official Python runtime as a parent image
FROM python:2.7-slim

# Set the working directory to /app
WORKDIR /app

# Copy the current directory contents into the container at /app
ADD . /app

# Install any needed packages specified in requirements.txt
RUN pip install --trusted-host pypi.python.org -r requirements.txt

# Make port 80 available to the world outside this container
EXPOSE 80

# Define environment variable
ENV NAME World

# Run app.py when the container launches
CMD ["python", "app.py"]
```

## Docker Services

Docker services are just containers in production--running one image but codifying the way it runs (ports, replicas, etc).

A `docker-compose.yml` file defines how Docker containers should behave in production. An app can be scaled by changing the number of replicas specified in the docker-compose file and re-running.

**Task**: a single container running in a service.

### Example docker-compose file
```
version: "3"
services: 
  # Run 5 instances of that image as a service called web.
  web: 
    image: deepkanwal/get-started:part-2
    deploy: 
      replicas: 5
      # Limit each replica to use, at most, 10% of the CPU (across all cores), and 50MB of RAM.
      resources:
        cpus: "0.1"
        memory: 50M
      # Immediately restart containers if one fails.
      restart_policy:
        condition: on-failure
    # Map port 80 on the host to web’s port 80.
    ports:
      - "80:80"
    # Instruct web’s containers to share port 80 via a load-balanced network called webnet. 
    networks:
      - webnet
# Define the webnet network with the default settings (which is a load-balanced overlay network).
networks:
  webnet:
```

### Common Commands

```
# List stacks or apps
docker stack ls

# Deploy a docker service with path to compose file with -c flag
docker stack deploy -c <composefile> <appname>

# List docker services running associated with an app
docker service ls

# Show running tasks associated with an app
docker service ps <service>

# Inspect task or container
docker inspect <task or container>

# Tear down an application
docker stack rm <appname>
```

## Swarms and Stacks

Docker by default runs in a single-host mode on a local machine. It can be switched to **swarm mode** to enable use of swarms. This converts the current machine to a swarm manager and executes commands on the swarm (rather than just the current machine. 

**Swarm**: A 'Dockerized' cluster (group of machines running docker) to allow for multi-container and multi-machine apps.

**Swarm Manager**: machines in the swarm that execute docker commands and authorize other machines to join.

**Worker**: Machines that only provide capacity. They do not have the authority to tell another machines what it can and can't do.

**Node**: machine in a Docker swarm. These can be physical or virtual.

Nodes in a swarm participate in an ingress **routing mesh**. This ensures that a service deployed at a certain port within your swarm always has the port reserved to itself no matter what node is running the container. 

![](https://docs.docker.com/engine/swarm/images/ingress-routing-mesh.png)
A routing mesh for a service called `my-web` published at port 8080 on a 3-node swarm.

**Stack**: A group of interrelated services that share dependencies and can be orchestrated and scaled together. 

New services can be added via the docker-compose.yml file. For example, adding Redis:

```
  redis:
    image: redis
    ports:
      - "6379:6379"
    volumes:
      - "/home/docker/data:/data"
    deploy:
      placement:
        constraints: [node.role == manager]
    command: redis-server --appendonly yes
    networks:
      - webnet
```

### Common Commands

```
# Create a VM (Mac, Win7, Linux)
docker-machine create --driver virtualbox myvm1 

# View basic information about your node
docker-machine env myvm1      
          
# Run a command on a node in your swarm
docker-machine ssh myvm1 "docker ..."         

# Open an SSH session with the VM; type "exit" to end
docker-machine ssh myvm1
   
# Make the worker leave the swarm
docker-machine ssh myvm2 "docker swarm leave" 

# Make master leave, kill swarm
docker-machine ssh myvm1 "docker swarm leave -f" 

# list VMs, asterisk shows which VM this shell is talking to
docker-machine ls 

# Start a VM that is currently not running
docker-machine start myvm1            

# Mac command to connect shell to myvm1
eval $(docker-machine env myvm1)         

# Deploy an app; command shell must be set to talk to manager (myvm1), uses local Compose file
docker stack deploy -c <file> <app> 

# Disconnect shell from VMs, use native docker
eval $(docker-machine env -u)     

# Stop all running VMs
docker-machine stop $(docker-machine ls -q)               

# Delete all VMs and their disk images
docker-machine rm $(docker-machine ls -q) 
```






