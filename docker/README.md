# INTRO

* __Docker client__ is the cli application that we use to manage and interact with the docker engine (specifically, with the docker daemon). Receives our cli commands, translates them for the docker daemon and submits them to it. 
* __Docker engine (Docker daemon + Containerd + Runc, ecc)__ - process that runs and manages containers, that receives the commands from the docker client and manages containers
    * `docker system info`
* __Images__
    * static (build-tyme) image of the container (containing all info about the container's user-space: filesystem + libraries, network/cpu/io configurations, ecc). Sort of a "static snapshot": alike to a Java class or a VM template, from which we can instantiate something)
    * An image is made of many read-only layers which are all used to compose the final image: image starts from a base layer, then every modification (defined in docker file commands) is saved into another layer (e.g. ubuntu + vim + update os + java). This is used by docker to realize a cache mechanism: it'll cache layers and use them when appropriate, building only not yet cached layers each time, thus improving performance
    * `docker inspect <IMAGE> | grep Layers`
* __Container__ 
    * instance (run-time) of a container image (specific instance, alike to Java object or VM instance). It depends on its image: it's not possible to delete an image if a related container is still running
    * a container executes until there is a process executing, i.e. until the start application (or any of its derived processes) is running (when it is over, it stops)
* __Dockerfile__
    * file that defines the configuration and run steps (layers) to define and construct a container image

# ONLINE PLAYGROUNDS

* [play-with-docker](https://www.docker.com/play-with-docker)
* [Katacoda](https://www.katacoda.com/)

# INSTALL

* [See docs](https://docs.docker.com/engine/install/ubuntu/)

# CLI BASICS

```bash
docker # see all sub-commands
docker [COMMAND] # see all sub-commands
docker [COMMAND] --help # help
docker run hello-world # check dockerhub connectivity
```

# SYSTEM INFO

```bash
# SYSTEM INFO
docker system info 
cat /etc/docker/daemon.json # daemon config file
```

# IMAGE PULLING

* __Registry (e.g. dockerhub, on-premise, etc) > Repositories (official (maintained) vs unofficial) > Images > Tags (versions)__ 
* `docker image pull <REPOSITORY>:<TAG>` (no tag specified = latest version)
    * Using tags should ensure that you are pulling the correct version of the image
    * if the image is already cached, it won't download it again
    * `docker run` - will execute the pulling (if the image is not available locally) before running
* __Digest__
    * Every time you pull an image, the image digest is returned. That hash is immutable for that specific image. You can save it and then control it with the one returned by a newly pulled image to ensure that the image hasn't change and it is exactly the one that returned your digest.
    * You can directly pull using digest: `docker pull <REPOSITORY>:<DIGEST>` (though this way the image will be tag-less)
* __Manifest.list__ = __Multi-Architecture compatibility file for an image:tag__ = Is a json file for each specific image:tag containing a list of the many architecture:os-specific versions of that image:tag, along with their compatibilities (architectures, os, mediatype, etc) and digests. Thanks to this file, Docker will take care of pulling the right image for your architecture:os automatically.

```bash
# PULL IMAGE
docker image pull <REPOSITORY>:<TAG> # TAG can be substituted with DIGEST
docker pull <REPOSITORY>:<TAG>

# PULL (if necessary) AND RUN IMAGE
docker run <REPOSITORY>:<TAG> # TAG or DIGEST
```

# IMAGE-MANAGEMENT

```bash
# HELP
docker image # manage images
docker images --help # list images

# SEARCK IN DOCKERHUB
docker search <NAME>

# PULL IMAGE
docker image pull <REPOSITORY>:<TAG> # TAG or DIGEST
docker pull <REPOSITORY>:<TAG>

# LIST IMAGES
docker image ls
docker images
docker images -q # only ids
docker images -f # filters

# INSPECT (low level info)
docker inspect <IMAGE[:TAG]>

# HISTORY (last operations on that image)
docker history <IMAGE[:TAG]>

# PRUNE (remove unused(dangling) images)
docker prune 

# RUN (instantiates a new container from the image)
docker run <IMAGE[:TAG]>

# REMOVE IMAGE
docker rmi <IMAGE>
docker image rm <IMAGE>
```

# CONTAINER MANAGEMENT

```bash
# LIST CONTAINERS
docker ps # running
docker container ls
docker ps -a # all
docker ps -a -f status=exited # filter exited
docker ps -q # ids
docker ps -a -f status=exited -q # ids of exited

# OUTPUT FORMATTING
docker ps --format '{{.Names}} container is using {{.Image}} image' # ps formatting
docker ps --format 'table {{.Names}}\t{{.Image}}' # tabular
docker ps -q | xargs docker inspect --format '{{.Id}} - {{.Name}} - {{.NetworkSettings.IPAddress}}' # inspect formatting

# REMOVE
docker rm <CONTAINER> # or ID
docker rm $(docker ps -a -f status=exited -q) # all exited
# it is a best practice to stop a container before removal

# PRUNE (remove all stopped)
docker container prune

# RUN ([pull +] instantiate from image)
docker container run <IMAGE>[ <COMMAND>]
docker run <IMAGE>[ <COMMAND>]
docker run --name="<NAME>" <IMAGE> # named
docker container run -it ubuntu /bin/bash # interactive tty
docker run -d <CONTAINER> <COMMAND> # background
docker container run -d -it ubuntu /bin/bash # background
docker run <IMAGE> sleep <SECONDS> # keep-alive
docker run -it --name myubuntu ubuntu /usr/bin/find / -iname '*.sh' # command + arguments (bash-wise)

# RUN WITH PORT-MAPPING
docker run -d -p<PHOST:PCONTAINER> # tcp port-mapping
docker run -d --publish <PHOST:PCONTAINER> # tcp port-mapping

# ATTACH TTY TO A RUNNING CONTAINER
docker attach <CONTAINER> # or ID

# DETACH (exit tty, container keeps executing)
CTRL+P CTRL+Q

# EXECUTE COMMAND INTO A RUNNING CONTAINER (from outside)
docker exec <CONTAINER> <COMMAND> # or ID
docker exec -it <CONTAINER> /bin/bash # open bash tty
docker attach <CONTAINER> # same as above

# SEE CONTAINER INFO
docker top <CONTAINER> # processes
docker stats <CONTAINER> # usage statistics
docker ps -q | xargs docker stats # usage stats (all)
docker container logs <CONTAINER> # logs

# SEE CONTAINER CHANGES IN RESPECT TO ITS ORIGINAL IMAGE
docker container diff <CONTAINER>

# STOP EXECUTION
docker stop <CONTAINER> # or ID

# START EXECUTION (of a stopped container)
docker start <CONTAINER> # or ID
```

# RESTART POLICIES

* `NO` (default) - exited containers won't be restarted
* `ALWAYS` - always restart, unless the container was explicitally stopped. On docker daemon restart it will be started (even if stopped).
* `UNLESS-STOPPED` - Same as above but on docker daemon restart these won't be started if stopped
* `ON-FAILED` - restart only when exited with non-zero code (error). On docker daemon restart will be started if stopped
* [more](https://docs.docker.com/engine/reference/run/#restart-policies---restart)

```bash
# RESTART POLICY --restart <POLICY>
docker run -d -it --restart always ubuntu /bin/bash
```

# DATA PERSISTENCE, VOLUMES AND FILESYSTEM MAPPING

* When stopping a container by default its data is preserved, until removal
    * __STORAGE DRIVERS__ (temporary storage) - component that manages the local disk space used by each container (to layerize and mount images and their filesystems, which is a temporary non-persistent space, used only during the lifecycle of the container and then removed). There are several types of storage drivers, with performance differences, and its choice is based on host, not the containers.
    * `docker system info | grep Storage\ Driver`
* __VOLUME__ (persistent storage) - volumes are persistent storage space decoupled from containers, which can be mounted in and used by a container. __It can be shared between different hosts, containers and third-party systems (e.g. cloud providers such as AWS, Azure, etc), and will persist on container removal__.
    1. Create a volume on a path of the host filesystem
    2. Create a container and mount that volume in a path of the container (or specify the mount path and the volume path in the container's image itself)
    3. The container path will refer to the host's volume and data inside that path will be in sync
    4. The same volume can be mounted on many containers and will persist on containers removal 
* __BIND-MOUNTS__ (host-dependent path mapping)
    * You can as well map a host filesystem path with a container path.
    * The result is similar but __the volume approach should be preferred: The host directory is, by its nature, host-dependent so bind mounts rely on the host machine’s filesystem having a specific directory structure available whereas volumes are managed by docker inside its directories__. For this reason, you can’t mount a host directory from Dockerfile because built images should be portable. A host directory wouldn’t be available on all potential hosts. - [more](https://docs.docker.com/storage/bind-mounts/#:~:text=Bind%20mounts%20have%20limited%20functionality,is%20mounted%20into%20a%20container.&text=By%20contrast%2C%20when%20you%20use,Docker%20manages%20that%20directory's%20contents.)

```bash
# VOLUMES
docker volume
docker volume --help

# CREATE
docker volume create <NAME>

# ASSOCIATE VOLUME WITH CONTAINER
docker run -v <VOLUME>:<CNTPATH> <IMAGE>[ <COMMAND>]
docker run -it -v myvolume:/test ubuntu /bin/bash
# if the volume doesn't exist, it'll be created
# if the container exist already, you can commit to a new image and then run -v that image
# by default the container has read-write access, but you can make it read-only
docker run -v <VOLUME>:<CNTPATH>:ro <IMAGE>[ <COMMAND>]

# SAME AS ANOTHER CONTAINER
docker run --volumes-from <CONTAINERB> <IMAGE>[ <COMMAND>]

# BIND MOUNTS (host-dependent)
docker run -it -v /host/path:/cnt/path ubuntu /bin/bash # abs path instead of volume name

# LIST
docker volume ls

# INSPECT
docker volume inspect <VOLUME>
# locate volume path
docker volume inspect <VOLUME> | grep Mountpoint 

# REMOVE
docker volume rm <VOLUME>

# PRUNE
docker volume prune
```

# COMMIT

* __Commit is taking a snapshot of the current state of a container (file changes, settings, etc) and saving it as an image__
    * __Generally, it is better to use Dockerfiles to manage your images in a documented and maintainable way__
    * Anyway it can be useful to allow you to debug a container by running an interactive shell, or to export a working dataset to another server, etc
    * This basically means if your "running" container are generating logs files, update packages, or making file changes, they will be saved into the image
    * To put it in a different analogy `docker build` (i.e. building a dockerfile) is like `git clone`, while `docker commit` (committing) is like `git commit`
* The commit operation will not include any data contained in volumes mounted inside the container.

```bash
docker commit --help

# COMMIT
docker commit <CONTAINER> [<REPOSITORY>[:<TAG>]]
docker commit --message <MESSAGE> --author <AUTHOR> <CONTAINER> [<REPOSITORY>[:<TAG>]]
```

# DOCKERFILE

* Dockerfiles are scripts that define and automate the steps to build a docker image
    * allow to define and maintain images in a documentented and maintainable way
    * allow non-authors to check what is inside an image (e.g. for security reasons), and/or customize it to suit their needs
    * automate the steps to build an image, which are defined in a clear and transparent way thus avoiding "golden images" (un-maintainable images which are not reproduceable because you no longer know how they were built, not easily understandable by other team members, not easily upgradable, etc). For this reason dockerfile approach is better than the commit approach for most cases
* It must be named `Dockerfile`
* It contains dockerfile commands (`FROM`, `CMD`, `EXPOSE`, `ENV`, `RUN`, `ENTRYPOINT`, `COPY`, `ADD`, etc) combined with shell commands (e.g. bash)
* It is built into an image by the docker daemon via the `docker build -f <DIRPATH>` command
* Best practices:
    * use `.dockerignore` file to exclude files from the building process
    * use versioning to better organize versions of the dockerfile
    * keep the images smallest as possible (install only what is necessary)
    * one application for container (thus improving decoupling and maintainibility), then make containers communicate through network
    * [more](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)
* [See docs](https://docs.docker.com/engine/reference/builder/)

Example:

```dockerfile
# Dockerfile example
FROM ubuntu:latest
LABEL maintainer="Me"
VOLUME myVolume
ENV MYHOME=/home/
RUN apt-get update
RUN apt-get -y install net-tools
RUN apt-get -y install netcat
ADD hello.sh /home/hello.sh
ADD hello1.sh /home/hello1.sh
RUN chmod 777 ${MYHOME}/hello.sh
RUN chmod 777 ${MYHOME}/hello1.sh
WORKDIR /home
CMD ./hello.sh
```

```bash
# BUILD example
docker build -f <DIRPATH> # of a diretory containing a Dockerfile
```

## Docker build

```bash
docker build --file <DOCKERFILE> --tag <REPOSITORY>:<TAG>
docker build -f <DOCKERFILE> -t <REPOSITORY>:<TAG>
docker build <DIRECTORY> # use Dockerfile inside the directory
```

## Dockerfile commands

* `FROM` - base image to use for inizialization (before any other change)
    * It is optional, the image can also be built from scratch but it is quite complex to do so: tools like DEBOOTSTRAP, YUMBOOSTRAP and RINSE are typically used to simplify this task
    * Starting from a base image is simpler and quite common, and should be preferred when possible (dockerhub provides base images for most of situations) 
* `LABEL` - key value pairs used as label (e.g. maintainer="Me")
* `RUN` - execute commands inside the container (in a new layer and thus creating a new image. E.g., it is often used for installing software packages)
* `COPY` - copy files from host filesystem into the container's 
* `ADD` - same as copy, but also supports 2 other sources: URLs and you can extract a tar file from the source directly into the destination
* `ENV` - define environment variables into the container
* `CMD` or `COMMAND` - sets default command and/or parameters to be used on `docker run` unless otherwise specified (it can be easily overwritten from CLI).
    * e.g. in `docker run -i -t ubuntu bash` the command is `bash`
* `WORKDIR` - initial container's directory, in which CMD will be executed
* `ENTRYPOINT` - configure the executable to be used on `docker run` (on which the command will be used)
    * it will not be ignored when the user specifies a command on `docker run` and it is not easily changeable, specific options such as `--entrypoint` must be used.
    * e.g. in `docker run -i -t ubuntu bash` the entrypoint `/bin/sh -c` (default)
    * It can be used also as a default command to which provide arguments with `CMD` (or at runtime with `docker run <IMAGE> <COMMAND>`)
* `USER` - sets the user name (or UID) and optionally the user group (or GID) to use when running the image and for any RUN, CMD and ENTRYPOINT instructions that follow it in the Dockerfile
* `VOLUME` - binds a volume to a container's filesystem path (See volumes)
* `EXPOSE` - sets the ports on which a container listens for connections (which can be overriden at runtime using the port mapping flag `docker run -p`)

Other commands are available, [see docs](https://docs.docker.com/engine/reference/builder/).

_Note: RUN, COPY and ADD create a new layer of the image: this is used by docker for a caching mechanism: it'll actually build only new layers, using the cached ones whenever possible (e.g. if we installed curl, for new containers it won't be necessary to install it again because docker will use the cached layer)_.

```dockerfile
# DOCKERFILE COMMANDS SYNTAX

# FROM
FROM [--platform=<platform>] <image> [AS <name>]
FROM [--platform=<platform>] <image>[:<tag>] [AS <name>]
FROM [--platform=<platform>] <image>[@<digest>] [AS <name>]

# LABEL
LABEL <key>=<value> <key>=<value> <key>=<value> ...
LABEL maintainer="SvenDowideit@home.org.au"
LABEL "com.example.vendor"="ACME Incorporated"
LABEL com.example.label-with-value="foo"
LABEL version="1.0"
LABEL description="This text illustrates \
that label-values can span multiple lines."
LABEL multi.label1="value1" multi.label2="value2" other="value3"

# RUN
# SHELL FORM (default is /bin/sh -c on Linux and cmd /S /C on Windows)
RUN <command>
# EXEC FORM
RUN ["executable", "param1", "param2"] (exec form)
# Examples
RUN ["/bin/bash", "-c", "echo hello"]
RUN /bin/bash -c 'source $HOME/.bashrc; echo $HOME'
RUN /bin/bash -c 'source $HOME/.bashrc; \
echo $HOME'

# ENTRYPOINT
# EXEC FORM
ENTRYPOINT ["executable", "param1", "param2"]
# SHELL FORM
ENTRYPOINT command param1 param2
# Examples
ENTRYPOINT ["top", "-b"]
ENTRYPOINT ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]

# CMD
# EXEC FORM (best)
CMD ["executable","param1","param2"]
# DEFAULT EXEC FORM
CMD ["param1","param2"]
# SHELL FORM
CMD command param1 param2
# Examples
CMD echo "This is a test." | wc -
CMD ["/usr/bin/wc","--help"]

# COPY
COPY [--chown=<user>:<group>] <src>... <dest>
COPY [--chown=<user>:<group>] ["<src>",... "<dest>"]
COPY hom* /mydir/
COPY hom?.txt /mydir/
COPY test.txt relativeDir/
COPY --chown=55:mygroup files* /somedir/

# ADD
ADD [--chown=<user>:<group>] <src>... <dest>
ADD [--chown=<user>:<group>] ["<src>",... "<dest>"]
# same syntax as COPY (see examples above)

# ENV
ENV <key> <value>
ENV <key>=<value> ...
ENV myName="John Doe" myDog=Rex\ The\ Dog \
    myCat=fluffy
ENV myName John Doe
ENV myDog Rex The Dog
ENV myCat fluffy

# WORKDIR
WORKDIR /path/to/workdir

# USER
USER <user>[:<group>]
USER <UID>[:<GID>]
USER patrick

# VOLUME
VOLUME ["/data"]
VOLUME /myvol

# EXPOSE
EXPOSE <port> [<port>/<protocol>...]
EXPOSE 80/udp
```

# NETWORKING (single-host)

* [See docs](https://docs.docker.com/network/)
* __CNM - Container Network Model__ - the network management model used by docker, realized via __libnetwork__ library
* Docker creates custom iptables-netfilter rules to realize custom virtual networks in order to provide different kind of connectivity for the host's containers
* Each container sees its network as any other host (IP, gateway, routing table, DNS, ecc)
* __Docker DNSs automatically resolves containers names (on the same network) to their IPs__, thus simplifying their management (`ping <CONTAINERNAME>` works if container is reachable). This is only available on custom networks, not the default bridge (in the default bridge the IP addresses must be used) - [more](https://docs.docker.com/config/containers/container-networking/#dns-services)
* Docker provides 3 types of network drivers by default (unless configured otherwise):
    * __BRIDGE__ (default)
        * __It's like connecting some containers and the host to a switch, which provides LAN L2 connectivity between them__. Actually it is a software bridge, i.e. L2-Link-Layer device (in this case not hw but virtualized through sw) which forwards L2 traffic.
        * __Containers are isolated from what's outside the bridge network__ (i.e. other not connected to that bridge network and the external world)
        * __The external network is accessible only via the host__ (i.e. outside-ward connectivity is possible through host's NAT, inwards connectivity is possible only if a public port on the host is mapped via port-mapping on a container port (host-public-IP:PORT > host-natted-IP:PORT > port-mapping > container-IP:PORT)
        * __Only applies to containers on the same Docker daemon host__
        * [more](https://docs.docker.com/network/bridge/)
    * __HOST__ 
        * __Use the host network interface directly__: the container shares the host’s networking namespace (it does not get its own IP-address allocated), ports exposed by the container are exposed on the external network (security concerns)
        * [more](https://docs.docker.com/network/host/)
    * __NULL__
        * __no connectivity, no IP, only loopback__
        * [more](https://docs.docker.com/network/none/)
* Using `docker network create` you can create custom networks, i.e. create a virtual network interface connected to the host with specific iptables-netfilter rules (thus with custom behaviour), using one of the custom network drivers provided by docker and customizing it with options.
    * `--subnet` - subnet addresses (should be specified or conflicts between other host networks not managed by docker may arise)
    * `--gateway` - network gateway address
    * `--ip-masq` - ip masquerading (i.e. source ip address is changed when a packet exits the custom network, NAT-wise)
    * `--icc` - enable/disable inter-container communication
    * `--ip` - give a specific IP to the custom network (e.g. to bind the container to a specific physical network interface of the host, to simulate the direct usage of the physical interface e.g. to expose services on the external network (security concerns))
        * docker default networks (e.g. bridge) have dynamic IP address, assigning them a fixed IP may result in conflicts (what if the chosen IP is occupied when docker starts?)
    * An alternative equivalent syntax exists: `docker network create -o com."docker.network.bridge.host_binding_ipv4"="192...." my-network`
    * [more](https://docs.docker.com/engine/reference/commandline/network_create/)

```bash
# NETWORK COMMANDS
docker network --help

# LIST DOCKER NETWORKS
docker network ls

# INSPECT NETWORK
docker network inspect <NETWORK>

# CONNECT CONTAINER TO A NETWORK
docker network connect <NETWORK> <CONTAINER>
docker run --network=<NETWORK> <IMAGE>[: <TAG>]

# RUN CONTAINER WITH PORT-MAPPING
docker run -d -p<PHOST:PCONTAINER> 
docker run -d --publish <PHOST:PCONTAINER>

# DISCONNECT CONTAINER FROM A NETWORK
docker network disconnect <NETWORK> <CONTAINER>

# CREATE NETWORK
docker network create <NETWORK> # default driver is bridge
docker network create --driver=bridge --subnet=192.168.0.0/16 br0
docker network create -d overlay \
  --subnet=192.168.10.0/25 \
  --subnet=192.168.20.0/25 \
  --gateway=192.168.10.100 \
  --gateway=192.168.20.100 \
  --aux-address="my-router=192.168.10.5" --aux-address="my-switch=192.168.10.6" \
  --aux-address="my-printer=192.168.20.5" --aux-address="my-nas=192.168.20.6" \
  my-multihost-network

# REMOVE NETWORK
docker network rm <NETWORK>
```

# DOCKER COMPOSE (Single-host-service container orchestration)

* Containers can communicate with each others through network interfaces. In microservice architectures having an application composed by multiple containers communicating with each other via network is the standard (e.g. orders database + clients database + production web server + development web server, each in its own container, all communicating with each other). Best practice is to have multiple containers, one for each micro-service
* __Docker compose simplyfies the creation and management (starting, stopping, scaling of containers) of an application composed by multiple containers__ (similar to what the dockerfile does for image creation, but for containers orchestration) - [more](https://docs.docker.com/compose/)
    * it is included in Windows and Mac version of Docker, but must be installed separately on linux - [more](https://docs.docker.com/compose/install/)
* By default Compose sets up a single network for your app. Each container for a service joins the default network and is both reachable by other containers on that network, and discoverable by them at a hostname identical to the container name. - [more](https://docs.docker.com/compose/networking/)
* When scaling a service in our application (i.e. replicating it into more containers), docker DNS will distribute requests between them performing a rudimental load balancing between "parallel" containers of the service (when performing port-mapping though only one host port can be mapped to a container, so you must specify a range e.g. `80-85:80`). Another option is to use a load balancer as service (e.g. nginx)

## Docker compose file

* `docker-compose.yml` YAML file defines the SERVICEs of the application and their configurations
    * [See reference](https://docs.docker.com/compose/compose-file/)

```yml
# docker-compose.yml
version: '3'
services:
  web:
    build: .
    dockerfile: Dockerfile
    ports: 
        - "8888:80" # HOSTPORT:CONTAINERPORT
  redis:
    image: redis
```

```bash
#Dockerfile
FROM python:3.7
WORKDIR /app
ADD requirements.txt /app/requirements.txt
RUN pip install -r reqiorements.txt
ADD app.py /app/app.py
EXPOSE 80
CMD ["python", "app.py"]
```

## Docker compose commands

```bash
docker-compose # help

# VALIDATE DOCKER COMPOSE FILE
docker-compose config

# BUILD
docker-compose build
docker-compose -f ~/path/docker-compose.yml build

# START SERVICES
# (automatically builds if necessary)
docker-compose up 
docker-compose -f ~/path/docker-compose.yml up
docker-compose up -d # daemon/detach (background)
docker-compose up --scale <SERVICE>=<NUM> # scale

# STOP SERVICES
docker-compose down
docker-compose -f ~/path/docker-compose.yml stop
```

# DOCKER SWARM (multi-host service container orchestration)

* Docker swarm simplyfies the creation, scaling and management of services scaled in a multi-host cluster in a declarative way:
    * it lets you declare hosts (running docker swarm and labelled as manager or workers), services (and their desired characteristics e.g. number of replicas, network and storage resources, ports exposed to the outside world, etc)
    * it takes care of the low-level management details of container orchestration to realize the service you've declared (in a similar way to what docker-compose does for single-host services)
        * it lets you change the configuration of a service at run-time without restarting it thanks to the multi-host replication
* [more](https://docs.docker.com/engine/swarm/key-concepts/)

## Docker swarm entities

* __STACK__ = group of services (containers) which work together to realize an application (alike to a docker-compose stack)
* __TASK__ = instance of a service (container) running in a node
* __NODES__ = instances of the Docker engine participating in the swarm ("hosts": physical computer with a single OS, VMs, cloud environments, etc)
    * __WORKERS__ = receive and execute tasks dispatched from manager nodes
    * __MANAGERS__ = perform the orchestration and cluster management required to maintain the desired state of the swarm. In a production environment, at least 5 managers are advised (they will elect a single leader to conduct orchestration tasks), but for simple applications a single manager may be enough 
        * receives the service definition (alike to docker-compose: number of replicas, network and storage resources, ports exposed to the outside world, etc)
        * contains a registry service, which is queried by workers to get the images for container instantiation
        * create, manage and dispatch tasks to the worker nodes
        * receive requests from the client and distribute them to the worker nodes (load-balancing)
            * The load balancing performed is basic and not much configurable: for more complex load-balancing Kubernetes is better
        * A manager is by default also a worker, but you can make it manager only
    * Managers and workers need to be able to communicate via network - [more](https://docs.docker.com/engine/swarm/swarm-tutorial/#the-ip-address-of-the-manager-machine)

## Docker swarm commands

[See tutorial](https://docs.docker.com/engine/swarm/swarm-tutorial/)

```bash
# MANAGER INITIAZATION (on MANAGER)
docker swarm init --advertise-addr <IP> # IP towards workers (internal)
# will output the command to be launched on a worker to join him to this swarm
# (comprehensive of a token for secure auth)

# WORKER INITIALIZATION (on WORKER)
# use the command given as output on manager initialization, which has the form:
docker swarm join --token <TOKEN> <IP:PORT>

# SWARM STATUS INFO (on MANAGER)
docker swarm info

# LIST NODES (on MANAGER)
docker node ls

# MANAGER LEAVING (on MANAGER)
docker swarm leave --force

# DEPLOY SERVICE (on MANAGER)
docker service create --replicas <N> --name <NAME> <IMAGE> [<COMMAND>]
docker service create --replicas 1 --name helloworld alpine ping docker.com

# LIST SERVICES (on MANAGER)
docker service ls

# INSPECT SERVICE (on MANAGER)
docker service inspect --pretty <SERVICE>

# SEE WHAT NODES ARE RUNNING A SERVICE (in this moment, it can change)
docker service ps <SERVICE>

# SEE SERVICES RUNNING IN A NODE (on any NODE)
docker ps

# SCALE SERVICE (on MANAGER)
docker service scale <SERVICE>=<NUMBER-OF-TASKS>

# REMOVE SERVICE (on MANAGER)
docker service rm <SERVICE>

# UPDATE (on MANAGER)
docker service create \
  --replicas 3 \
  --name redis \
  --update-delay 10s \ # automatic rolling updates
  redis:3.0.6
# if your update failed, then the update status of the service is "paused"
docker service inspect --pretty <SERVICE>
# if it's paused, you can restart it with:
docker service update --image redis:3.0.7 redis
redis
# then check the rolling update
docker service inspect --pretty <SERVICE>
docker service ps <SERVICE-ID>
```

## Set node availability to Drain

* "Drain" status = do not receive tasks, stop running tasks. The manager launches replicas on other active nodes to replace the stopping ones. (only applies to swarm related containers)

```bash
# SET NODE AVAILABILITY TO DRAIN (on MANAGER)
docker node update --availability drain <WORKER>
docker node ls # Availability = Drain
docker node inspect --pretty <WORKER> # Status.Availability = Drain
docker service ps redis # check the updated task assignments

# REVERT NODE AVAILABILITY TO ACTIVE (on MANAGER)
docker node update --availability active <NODE-ID>
docker node ls # Availability = Active
docker node inspect --pretty <WORKER> # Status.Availability = Drain
docker service ps redis # check the updated task assignments
```

## Routing mesh

![](img/routing-mesh.png)

* Routing mesh is a functionality of docker swarm that simplifies the routing and load balancing of incoming requests.
* With routing mesh, each node will have a load balancer, so the request can now be received by any node indifferently
* Each node in the swarm will accept  connections on published ports for any service of the swarm, even if there’s no task running on that node, taking care of routing it to available nodes that have the corresponding active task
* You can configure routing mesh in many different ways: set up TCP-only, UDP-only or TCP+UDP connections, configure an external load balancer, and/or bypass the routing mesh, etc
* [more](https://docs.docker.com/engine/swarm/ingress/)

``` bash
# On service creation
docker service create \ 
  --name <SERVICE-NAME> \
  --publish published=<PUBLISHED-PORT>,target=<CONTAINER-PORT> \
  <IMAGE>
# On existing service
docker service update \
  --publish-add published=<PUBLISHED-PORT>,target=<CONTAINER-PORT> \
  <SERVICE>
# Inspect
docker service inspect --format="{{json .Endpoint.Spec.Ports}}" my-web
```

## Deploy docker swarm stacks (docker-compose deploy + docker stack)

* It is an extension of docker-compose, which adds the `deploy` section to the docker-compose.yml file to define swarm-related configurations (needs docker-compose and docker-swarm installed)
    * [deploy section reference](https://docs.docker.com/compose/compose-file/#deploy)
* [more](https://docs.docker.com/engine/swarm/stack-deploy/)

```yml
# docker-compose.yml
version: "3"
services:
  cluster1:
    image: nginx
    ports:
      - "8080:80"
    deploy: # service swarm settings
      replicas: 6
      restart_policy:
        condition: on-failure
      placement:
        constraints:
           - node.role == worker
  cluster2:
    image: redis
    ports:
      - "8081:80"
    deploy: # service swarm settings
      replicas: 6
      restart_policy:
        condition: on-failure
      placement:
        constraints:
           - node.role == worker
  cluster3: 
    image: httpd
    ports:
      - "8082:80"
    deploy: # service swarm settings
      replicas: 6
      restart_policy:
        condition: on-failure
      # no placement = on any node
```

```bash
# DEPLOY STACK (on MANAGER)
docker stack deploy \
    --compose-file=docker-compose.yml \
    <STACK>

# LIST STACKS
docker stack ls

# SEE SERVICES INFO
docker stack services <STACK>
# scale service
docker service scale mystack_myservice=2

# SEE CONTAINERS OF THE STACK
docker stack ps <STACK>

# REMOVE STACK
docker stack rm <STACK>
```

## Overlay networks (multi-host VXLAN L2 tunnelling)

* The `overlay` network driver creates a distributed network among multiple Docker daemon hosts. This network sits on top of (overlays) the host-specific networks, allowing containers connected to it (including swarm service containers) to communicate securely when encryption is enabled through VXLAN L2 tunnelling. Docker transparently handles routing of each packet to and from the correct Docker daemon host and the correct destination container.
  * Creates a VXLAN (encrypted L2 tunnelling), which means that containers inside that network will share a common private IP addressing space, like if they were on the same LAN (e.g. 10.0.0.0/24), thus they will be able to directly communicate with each other securely and directly
  * This simulates having the containers on the same host, decoupling the real network infrastructure from the virtual network used by containers, defined by docker
* [more](https://docs.docker.com/network/overlay/)

```bash
# CREATE OVERLAY NETWORK
docker network create -d overlay <NETWORK>
docker network ls # check
docker stack deploy --compose-file=<FILE> <STACK> # test
docker network inspect <NETWORK> # inspect
```

# DOCKER SECURITY BEST PRACTICES

* In docker, we can't estabilish a container accessing hierarchy: who has root access to the host can execute any command inside the containers (this is a downside to containers in respect to VMs, where we can controll access permissions more accurately)
  * we must be careful to who can access the docker host
  * we must maintain other services outside of that host 
* Images are not transparent and could execute any code: we should always use images defined by a dockerfile and check what it does, and possibly only use official ones (officially maintained by a trusted source)
* Consider using read only flags
  * Mount the container’s root filesystem as read only with `docker run ... --read-only` to limit the disk space writing of possible attackers (though some services need to write files/logs etc, so this must be considered separately for each case)
  * Mount volumes using the `:ro` option to make them read-only (`docker run -v <VOLUME>:<CNTPATH>:ro <IMAGE>[ <COMMAND>]` (same considerations as above)
* In any moment, we can check what changes has been made on a container in respect to its original image, with `docker container diff <CONTAINER>`
* [Katacoda playgrounds about docker security](https://www.katacoda.com/courses/docker-security)

## CIS & Docker Benchmark Security Tool

* CIS (Center for Internet Security) has released a pdf containing all crytical security aspects about docker - [more](https://www.cisecurity.org/benchmark/docker/)
* `docker benchmark security` is a tool (actually, a container that becomes part of our infrastructure) that executes automated tests regarding security checks on our infrastructure (checks config files, container runtimes, versions, security operations, etc). Info about Warnings resulting from the tests can be found on the pdf - [more](https://github.com/docker/docker-bench-security)

## Quay

* Quay is a registry, similar to dockerhub, which additionally performs security checks on every pushing/building of an image

# BEST PRACTICES

1. Isolation = one service only for each container
2. Images should contain only what is strictly necessary for the service to accomplish its tasks (thus images should be minimal, as small as possible)
3. Always secure the host root access (who has root access to the host can execute any command inside the containers, containers doesn't provide any access control mechanisms)
4. Always use last softwares available, for security patches (update docker and container's libraries). Consider using enterprise version instead of community version for additional security features and professional support for crytical projects

