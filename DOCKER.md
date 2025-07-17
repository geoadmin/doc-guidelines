# Docker

- [1. Base images](#1-base-images)
- [2 Best practices](#2-best-practices)
  - [Do not run as “root”](#do-not-run-as-root)
  - [Ordering of layers](#ordering-of-layers)
  - [Limit number of layers](#limit-number-of-layers)
  - [Sort multi-line arguments](#sort-multi-line-arguments)
  - [Use multistage builds](#use-multistage-builds)
- [Recipies](#recipies)
  - [Create unprivileged user with host user id](#create-unprivileged-user-with-host-user-id)

## 1. Base images

Whenever possible, official docker images based on `bookworm-slim` should be used (e.g. `python:3.13-bookworm-slim`, `nginx:1.19`, ...). Customized base and application images should be based on `bookworm-slim`.

## 2 Best practices

There’s plenty of sources on this topic, e.g.

- [https://docs.docker.com/develop/develop-images/dockerfile_best-practices/](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)
- [https://snyk.io/blog/10-docker-image-security-best-practices/](https://snyk.io/blog/10-docker-image-security-best-practices/)
- [https://cloud.google.com/solutions/best-practices-for-building-containers](https://cloud.google.com/solutions/best-practices-for-building-containers)
And specifically for python
- [https://pythonspeed.com/docker/](https://pythonspeed.com/docker/)
- [https://pythonspeed.com/articles/dockerizing-python-is-hard/](https://pythonspeed.com/articles/dockerizing-python-is-hard/)

The most important ones are reproduced here for reference:

- [1. Base images](#1-base-images)
- [2 Best practices](#2-best-practices)
  - [Do not run as “root”](#do-not-run-as-root)
  - [Ordering of layers](#ordering-of-layers)
  - [Limit number of layers](#limit-number-of-layers)
  - [Sort multi-line arguments](#sort-multi-line-arguments)
  - [Use multistage builds](#use-multistage-builds)
- [Recipies](#recipies)
  - [Create unprivileged user with host user id](#create-unprivileged-user-with-host-user-id)

### Do not run as “root”

This is a good practice in general and required by our Kubernetes cluster setup. The user can be changed with the `USER` instruction and requires that the user exists and has the correct permissions to access the relevant files. A minimal example could look like this

```Dockerfile
FROM python
RUN mkdir /app
RUN groupadd -r web && useradd -r -s /bin/false -g web web
WORKDIR /app
COPY index.html /app
RUN chown -R web:web /app
USER web
EXPOSE 8080
CMD ["python", "-m", "http.server", "8080"]
```

### Ordering of layers

The build process can considerably be speed up if the layers are ordered in good way, starting with layers that change least frequently (typically the base image mentioned with `FROM`) down to the layer that changes most frequently (e.g. the `COPY` command copying the source files in the container). If the image is constructed like this, a change in the code just requires one layer to be rebuilt.

```Dockerfile

FROM debian-bookworm              > the base layer

RUN apt-get update && \         \
    apt-get install -y \         |
        gosu \                    > the 'system' layer
        git \                    |
        vim                     /

COPY index.html /app            > the 'code' layer
```

### Limit number of layers

Each `RUN`, `CMD`, `FROM`, `COPY`, `ADD`... instruction creates a new layer. The number of layers can be e.g. be reduced by chaining (AND-ing) shell commands: `RUN apt-get update && apt-get install -y git`

### Sort multi-line arguments

Whenever possible, ease later changes by sorting multi-line arguments alphanumerically. This helps to avoid duplication of packages and make the list much easier to update. This also makes PRs a lot easier to read and review. Adding a space before a backslash (`\`) helps as well.

Here’s an example from the buildpack-deps image:

```Dockerfile
RUN apt-get update && apt-get install -y \
    bzr \
    cvs \
    git \
    mercurial \
    subversion
```

### Use multistage builds

[Multi-stage builds](https://docs.docker.com/develop/develop-images/multistage-build/) allow you to drastically reduce the size of your final image, without struggling to reduce the number of intermediate layers and files.

Because an image is built during the final stage of the build process, you can minimize image layers by leveraging build cache.

For example, if your build contains several layers, you can order them from the less frequently changed (to ensure the build cache is reusable) to the more frequently changed:

- Install tools you need to build your application
- Install or update library dependencies
- Generate your application

A Dockerfile for a Go application could look like:

```Dockerfile
FROM golang:1.11-alpine AS build

# Install tools required for project
# Run `docker build --no-cache .` to update dependencies
RUN apk add --no-cache git
RUN go get github.com/golang/dep/cmd/dep

# List project dependencies with Gopkg.toml and Gopkg.lock
# These layers are only re-built when Gopkg files are updated
COPY Gopkg.lock Gopkg.toml /go/src/project/
WORKDIR /go/src/project/
# Install library dependencies
RUN dep ensure -vendor-only

# Copy the entire project and build it
# This layer is rebuilt when a file changes in the project directory
COPY . /go/src/project/
RUN go build -o /bin/project

# This results in a single layer image
FROM scratch
COPY --from=build /bin/project /bin/project
ENTRYPOINT ["/bin/project"]
CMD ["--help"]
```

## Recipies

### Create unprivileged user with host user id

Sometimes it's useful or necessary to use an unprivileged user in the container that has the same uid as the host user executing the container. This is e.g. the case if you mount a volume and have the container create files. This can be achieved with the following snippet:

```Dockerfile
...
# Create unprivileged user
# # gosu is used to create an unprivileged user
# at `docker run`-time with the same uid as the host user. Thus, the mounted
# host volume has the correct uid:guid permissions. For details:
# https://denibertovic.com/posts/handling-permissions-with-docker-volumes/
# Installation of gosu:
# https://github.com/tianon/gosu/blob/master/INSTALL.md
RUN set -eux; \
        apt-get update; \
        apt-get install -y \
        gosu \
        rm -rf /var/lib/apt/lists/*; \
# verify that the binary works
        gosu nobody true
```
