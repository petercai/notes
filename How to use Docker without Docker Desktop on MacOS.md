# How to use Docker without Docker Desktop on MacOS
![](https://how.wtf/images/lLFsL5.webp)

Docker Desktop was my de facto choice on MacOS due to its ease of installation and container management. This changed after Docker updated their [subscription service agreement](https://www.docker.com/blog/updating-product-subscriptions/).

Step-by-step guide to installing Docker without Docker Desktop
--------------------------------------------------------------

The following tutorial assumes that you use `brew` as your package manager.

### Install docker

Firstly, install `docker` and `docker-credential-helper`.

```
1brew install docker-credential-helper docker
```

`docker-credential-helper` provides a way for Docker to use the MacOS Keychain as a credential store.

### Install colima

The true power comes from [colima](https://github.com/abiosoft/colima): a container runtime for MacOS and Linux.

Install it using `brew`:

### Start colima

Colima boasts its CLI ease of use! To get started, simply start the service:

### Using colima

After `colima` is installed, `docker` should hopefully work out-of-the-box:

```
1docker ps 2CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

Some applications do not respect `docker` contexts and will yield the following error:

```
1Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?
```

To remediate the issue, set the `DOCKER_HOST` variable.

```
1export DOCKER_HOST=unix://${HOME}/.colima/default/docker.sock
```

[#mac](https://how.wtf/tags/mac.html)  

[Reply to this post by email ↪](mailto:thomas@how.wtf?subject=Reply%20to%20%22How%20to%20use%20Docker%20without%20Docker%20Desktop%20on%20MacOS%22)