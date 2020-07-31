# INTRO

* __Docker client__ is the cli application that we use to manage and interact with the docker engine (specifically, with the docker daemon). Receives our cli commands, translates them for the docker daemon and submits them to it. 
* __Docker engine (Docker daemon + Containerd + Runc, ecc)__ - process that runs and manages containers, that receives the commands from the docker client and manages containers
    * `docker system info`
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
* [See docs](https://docs.docker.com/engine/reference/builder/)
* [best practices](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)

## DOCKERFILE COMMANDS

* `FROM` - base image to use for inizialization (before any other change). It is optional, the image can also be built from scratch.
* `LABEL` - key value pairs used as label (e.g. maintainer="Me")
* `RUN` - execute commands inside the container (in a new layer and thus creating a new image. E.g., it is often used for installing software packages)
* `COPY` - copy files from host filesystem into the container's 
* `ADD` - same as copy, but also supports 2 other sources: URLs and you can extract a tar file from the source directly into the destination
* `ENV` - define environment variables into the container
* `CMD` or `COMMAND` - sets default command and/or parameters to be used on `docker run` unless otherwise specified (it can be easily overwritten from CLI).
    * e.g. in `docker run -i -t ubuntu bash` the command is `bash`
* `WORKDIR` - initial container's directory, in which CMD will be executed
* `ENTRYPOINT` - configure the executable to be used on `docker run` (on which the command will be used): it will not be ignored when the user specifies a command on `docker run` and it is not easily changeable, specific options such as `--entrypoint` must be used.
    * e.g. in `docker run -i -t ubuntu bash` the entrypoint `/bin/sh -c` (default), which is 
* `USER` - sets the user name (or UID) and optionally the user group (or GID) to use when running the image and for any RUN, CMD and ENTRYPOINT instructions that follow it in the Dockerfile
* `VOLUME` - binds a volume to a container's filesystem path (See volumes)
* `EXPOSE` - sets the ports on which a container listens for connections (which can be overriden at runtime using the port mapping flag `docker run -p`)

Other commands are available, [see docs](https://docs.docker.com/engine/reference/builder/).

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
