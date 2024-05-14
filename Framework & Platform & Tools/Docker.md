
# Docker

### Concepts

- **container**: is a sandboxed process on your machine that is isolated from all other processes on the host machine.
- **container image**: provides a cuntomized filesystem for container (all dependencies, configuration, scripts, binaries, environment variables, etc.)
- **registry**: The Registry is a stateless, highly scalable **server side application** that stores and lets you distribute Docker images. 

### CLI Commands

- `docker ps`: list running containers, add `-a` for all containers

- `docker build`: build an image from a *Dockerfile*

- `docker create`: create container from image

- `docker start`: start existing container

- `docker run`: create + start

  - no network isolation: `--network host`

   > for example, ssh is default to port 22. in host networking mode, set the port to another port, then the docker container's ssh service would be running directly on that port on host machine.
   > https://docs.docker.com/network/network-tutorial-host/

- `docker exec`: Run a command in a running container

  - `docker exec -it <container_id> sh`: get into container's shell (NOT SSH!)
  - `docker exec -it <container_id> /bin/bash`: can be used to start multiple cli
  - `docker exec <container_id> <command>`: run command

- `docker inspect`: check detail of a container/image

- view docker file of image with full-length info: 
  `docker history <image-name>:<image-tag> --no-trunc=true`

- save current container as new image: `docker commit`

- `docker stats`: real-time resources usage, use ` --no-stream` to print instant usage

- `docker image`: use `ls` to list, `rm` to remove

- `docker push IMG`: push a image or repo to a registry

- `docker pull IMG`: pull a image or repo from a registry

- `docker tag SRC_IMG[:TAG] TAR_IMG[:TAG]`: create a tag TAR_IMG refers to SRC_IMG

- `docker cp src_file container_id:/dst/path`: copy file to container


### Dockerfile
Docker can build images automatically by reading the instructions from a textfile named Dockerfile.
- `FROM`: Initializes a new build stage and sets the *Base Image* for subsequent instructions. The image can be any valid image. A valid Dockerfile must start with a `FROM` instruction.

- `RUN`: Execute any commands in a new layer on top of the current image and commit the results. The resulting committed image will be used for the next step in the Dockerfile.

- `CMD`: The command the container executes by default when you launch the built image (e.g. bash). Details [here](https://docs.docker.com/engine/reference/builder/#cmd).
  > `CMD` can be overridden when starting a container with `docker run $image $other_command`. 
  
- `ENTYRPOINT`: Sets the process that’s executed when your container starts. If not set, docker defaults to using `/bin/sh -c`.
  > `ENTRYPOINT` can be changed using the `--entrypoint` flag, but this should rarely be necessary. If you do change it, you’ll almost certainly need to set a custom `CMD` too.
  
- `ADD`: Copies new files, directories or remote file URLs from `<src>` and adds them to the filesystem of the image at the path `<dest>`. Allows 3 sources: 
  
  1. local file or directory
  2. URL
  3. compression format (e.g. gzip, bzip2, xz), extract automatically
  
- `COPY`: copies new files or directories from <src> and adds them to the filesystem of the container at the path <dest>. 
  > `COPY` only allows copying in a local or directory from your host (the machine-building the Docker image) into the Docker image.
  
- `ENV`: Sets the environment variable to a value.

- `ARG`: Defines a variable that users can pass at build-time to the builder with the `docker build` command using the `--build-arg <varname>=<value>` flag. An `ARG` instruction can optionally include a default value. 会被 `--build-arg` 传入的值覆盖

- `EXPOSE`: Informs Docker that the container listens on the specified network ports at runtime.
  > It does not actually publish the port. It functions as a type of documentation between the person who builds the image and the person who runs the container, about which ports are intended to be published. To actually publish the port when running the container, use the `-p` or `-P` flag on `docker run`.
  
- `USER`: Sets the user name (or UID) and optionally the user group (or GID) to use as the default user and group for the remainder of the current stage.

- `WORKDIR`: Sets the working directory for any `RUN`, `CMD`, `ENTRYPOINT`, `COPY` and `ADD` instructions.

- `LABEL`: Add metadata to an image.



# Containerd

docker 也是基于 containerd 做一定封装，

### ctr

- 有命名空间 `namespace` 概念，k8s的镜像默认在 `k8s.io` 下
- `ctr -n k8s.io image export`: 导出镜像
- `ctr -n k8s.io image import`: 导入镜像

### crictl
