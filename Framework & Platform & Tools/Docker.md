Docker Daemon

Docker CLI

### Docker CLI Commands

- `docker build`: build an image from a Dockerfile
- `docker run`: Run a command in a new container
- `docker exec`: Run a command in a running container
  - `docker exec -it <container_id> sh`: get into container's shell
  - `docker exec -it <container_id> /bin/bash`: can be used to start multiple cli
  - `docker exec <container_id> <command>`: run command
- `docker ps`: list all running containers
- `docker tag SRC_IMG[:TAG] TAR_IMG[:TAG]`: create a tag TAR_IMG refers to SRC_IMG
- `docker push IMG`: push a image or repo to a registry
- `docker pull IMG`: pull a image or repo from a registry
- `docker image`: manage a image
- xxx

### Docker Regisry

The Registry is a stateless, highly scalable **server side application** that stores and lets you distribute Docker images. 

For example, this command starts a container which works as a registry: `docker run -d -p 5000:5000 --name registry registry:2`

### Terminology

- **container**: is a sandboxed process on your machine that is isolated from all other processes on the host machine.
- **container image**: provides a cuntomized filesystem for container (all dependencies, configuration, scripts, binaries, environment variables, etc.)
- ​