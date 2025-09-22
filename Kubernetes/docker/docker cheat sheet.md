
# Docker CLI cheatsheet

Here’s a handy **Docker CLI cheatsheet** to help you work efficiently with Docker:

---

### 🐳 **Docker Basics**

|Command|Description|
|---|---|
|`docker version`|Show Docker version info|
|`docker info`|Display system-wide information|
|`docker help`|Show help for Docker commands|

---

### 📦 **Images**

|Command|Description|
|---|---|
|`docker search python`|Search Docker Hub for images|
|`docker pull python`|Download an image|
|`docker images`|List downloaded images|
|`docker rmi <image>`|Remove an image|

---

### 🏗️ **Containers**

|Command|Description|
|---|---|
|`docker run -it python`|Run a container interactively|
|`docker run -d -p 8000:8000 python`|Run in detached mode with port mapping|
|`docker ps`|List running containers|
|`docker ps -a`|List all containers|
|`docker stop <container>`|Stop a running container|
|`docker rm <container>`|Remove a container|

---

### 🛠️ **Build & Manage**

|Command|Description|
|---|---|
|`docker build -t myapp .`|Build image from Dockerfile|
|`docker exec -it <container> bash`|Run a command inside a container|
|`docker logs <container>`|View container logs|
|`docker cp <container>:/path /host/path`|Copy files from container to host|

---

### 🧹 **Cleanup**

|Command|Description|
|---|---|
|`docker system prune`|Remove unused data|
|`docker image prune`|Remove unused images|
|`docker container prune`|Remove stopped containers|

---

Would you like this as a downloadable file or a printable PDF?

# list docker command cheat sheet

Here is a concise Docker command cheat sheet covering the most commonly used commands for managing images, containers, and the Docker environment:

## Docker Command Cheat Sheet

### General Commands

- `docker version`
Show Docker client and server version info.
- `docker info`
Display system-wide information about Docker.
- `docker help`
Show help for Docker CLI.
- `docker <command> --help`
Show help for a specific Docker command.


### Image Management

- `docker build -t <image_name>:<tag> .`
Build an image from a Dockerfile in the current directory.
- `docker images`
List all local Docker images.
- `docker rmi <image_name_or_id>`
Remove an image.
- `docker pull <image_name>`
Pull an image from a registry.
- `docker push <image_name>`
Push an image to a registry.


### Container Management

- `docker run -it --name <container_name> <image_name> [command]`
Run a container interactively with a name.
- `docker run -d -p <host_port>:<container_port> <image_name>`
Run a container in detached mode with port mapping.
- `docker ps`
List running containers.
- `docker ps -a`
List all containers (including stopped).
- `docker stop <container_id_or_name>`
Stop a running container.
- `docker start <container_id_or_name>`
Start a stopped container.
- `docker restart <container_id_or_name>`
Restart a container.
- `docker rm <container_id_or_name>`
Remove a stopped container.
- `docker exec -it <container_id_or_name> /bin/sh`
Open an interactive shell inside a running container.
- `docker logs <container_id_or_name>`
Show logs of a container.


### Container Resource and System Management

- `docker stats`
Display live resource usage stats for running containers.
- `docker system df`
Show disk usage by Docker.
- `docker system prune -af`
Remove all unused containers, networks, images (both dangling and unreferenced), and optionally volumes.


### Network and Volume

- `docker network ls`
List Docker networks.
- `docker volume ls`
List Docker volumes.


### Docker Compose (for multi-container apps)

- `docker-compose up`
Start services defined in docker-compose.yml.
- `docker-compose down`
Stop and remove containers, networks, volumes created by `up`.
- `docker-compose ps`
List containers managed by Compose.
- `docker-compose logs`
View logs for Compose services.

---

For a detailed and printable version, you can refer to the official Docker CLI cheat sheet PDF from Docker Docs or comprehensive online cheat sheets such as those by Collabnix or Spacelift[^1][^2][^3].

This cheat sheet covers essential commands to efficiently work with Docker in development and troubleshooting.

<div style="text-align: center">⁂</div>

[^1]: https://docs.docker.com/get-started/docker_cheatsheet.pdf

[^2]: https://dockerlabs.collabnix.com/docker/cheatsheet/

[^3]: https://spacelift.io/blog/docker-commands-cheat-sheet

[^4]: https://www.reddit.com/r/docker/comments/17zldvu/the_ultimate_docker_cheat_sheet/

[^5]: https://pagertree.com/blog/docker-commands-cheat-sheet

[^6]: https://www.docker.com/resources/cli-cheat-sheet/

[^7]: https://kapeli.com/cheat_sheets/Dockerfile.docset/Contents/Resources/Documents/index

# docker build

When building Docker images, you can increase the verbosity of the output to help with debugging or understanding what's happening during the build process. Here are several ways to make `docker build` more verbose:

---

### 🛠️ **1. Use `--progress=plain` (Docker BuildKit)**

If you're using Docker BuildKit (enabled by default in recent Docker versions), this option shows detailed logs:

```Shell
DOCKER_BUILDKIT=1 docker build --progress=plain -t my-image .
```

---

### 🧱 **2. Disable BuildKit (for classic builder output)**

If you prefer the older, more verbose output format:

```Shell
DOCKER_BUILDKIT=0 docker build -t my-image .
```

This can sometimes give more granular step-by-step output, especially if you're troubleshooting.

---

### 🔍 **3. Add `RUN` commands with `-x` or `set -ex`**

Inside your Dockerfile, you can make shell commands verbose:

```Dockerfile
RUN set -ex && \
    echo "Installing dependencies..." && \
    apt-get update && \
    apt-get install -y curl
```

- `set -e`: Exit on error.
- `set -x`: Print each command before executing it.

---

### 🧪 **4. Use `--no-cache` to force all steps to run**

This ensures Docker doesn't skip steps due to caching:

```Shell
docker build --no-cache -t my-image .
```

---

### 🧰 **5. Combine for maximum verbosity**

```Shell
DOCKER_BUILDKIT=1 docker build --progress=plain --no-cache -t my-image .
```

---

Would you like help debugging a specific Docker build or Dockerfile? I can help analyze logs or optimize your Dockerfile too.
