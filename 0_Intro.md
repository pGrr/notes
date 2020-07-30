# Docker

* __Docker client__ is the cli application that we use to manage and interact with the docker engine (specifically, with the docker daemon). Receives our cli commands, translates them for the docker daemon and submits them to it. 
* __Docker engine (Docker daemon + Containerd + Runc, ecc)__ - process that runs and manages containers, that receives the commands from the docker client and manages containers
* __Images__
    * static (build-tyme) image of the container (containing all info about the container's user-space: filesystem + libraries, ecc). Sort of a "static snapshot": alike to a Java class or a VM template, from which we can instantiate something)
    * An image is made of many read-only layers which are all used to compose the final image: image starts from a base layer, then every modification (defined in docker file commands) is saved into another layer (e.g. ubuntu + vim + update os + java) - `docker inspect <IMAGE>` ("Layers") 
* __Container__ 
    * instance (run-time) of a container image (specific instance, alike to Java object or VM instance). It depends on its image: it's not possible to delete an image if a related container is still running
* __Dockerfile__
    * file that defines the configuration and run steps (layers) to define and construct a container image

## Online interactive docker playgrounds

play-with-docker, Katacoda. 

# Image pulling

* __Registry (e.g. dockerhub, on-premise, etc) > Repositories (official (maintained) vs unofficial) > Images > Tags (versions)__ 
* `docker image pull <REPOSITORY>:<TAG>` (no tag specified = latest version)
    * if the image is already cached, it won't download it again
    * `docker run` - will execute the pulling (if the image is not available locally) before running