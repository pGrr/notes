# INTRO

* __Docker client__ is the cli application that we use to manage and interact with the docker engine (specifically, with the docker daemon). Receives our cli commands, translates them for the docker daemon and submits them to it. 
* __Docker engine (Docker daemon + Containerd + Runc, ecc)__ - process that runs and manages containers, that receives the commands from the docker client and manages containers
* __Images__
    * static (build-tyme) image of the container (containing all info about the container's user-space: filesystem + libraries, network/cpu/io configurations, ecc). Sort of a "static snapshot": alike to a Java class or a VM template, from which we can instantiate something)
    * An image is made of many read-only layers which are all used to compose the final image: image starts from a base layer, then every modification (defined in docker file commands) is saved into another layer (e.g. ubuntu + vim + update os + java) - `docker inspect <IMAGE>` ("Layers") 
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
docker run -d -p<PHOST:PCONTAINER> # tcp port-mapping
docker run <IMAGE> sleep <SECONDS> # keep-alive
docker run -it --name myubuntu ubuntu /usr/bin/find / -iname '*.sh' # command + arguments (bash-wise)

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

# STOP EXECUTION
docker stop <CONTAINER> # or ID

# START EXECUTION (of a stopped container)
docker start <CONTAINER> # or ID
```

# DATA PERSISTENCE AND VOLUMES

* When stopping a container its data is preserved, until removal

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