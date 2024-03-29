# docker-utils
Docker util commands

## Dockerfile

Remember docker images are built by layers, one on top of the other, so it is recommended to think carefully in which order to put them so you take the most out of the cache system.

For example, if you want to run a node container, a good idea would be to structure your dockerfile as follows

```Dockerfile
...
COPY package.json package-lock.json /usr/src/
RUN npm install --only=production

COPY build/ /usr/src/
ENTRYPOINT ["node", "/usr/src/main.js"]
```

This way if you change anything in your build directory and want to rebuild the image, Docker won't install npm packages again, it'll use the cached layer.

However, if you change your `package.json` or `package-lock.json`, that cache will be invalidated and npm packages will be installed again

## Container management

**Stop all containers**
```shell
docker stop $(docker ps -q)
```

**Delete unused containers**

WARNING: this will delete all containers that are not up and its information will be deleted (unless you're using volumes)
```shell
docker container prune
```

## Image management

**Delete unused images**
WARNING: this will delete all images that are not being used (ie a container with that images doesn't exist)

```shell
docker image prune
```

**Export image** to tar file

```shell
docker image save image_name > image_name.img.tar
```

**Image load** from tar file

```shell
docker image load < image_name.img.tar
```

## Exec vs shell form

**Exec form**: is NOT executed inside a shell

```cmd
ENTRYPOINT ["node", "/usr/src/main.js"]
```

Will be executed as `node /usr/src/main.js`

**Shell form**: IS executed inside a shell

```cmd
ENTRYPOINT node /usr/src/main.js
```

Will be executed as `/bin/sh -c node /usr/src/main.js`

## Docker in docker

Mount the docker's UNIX socket from host machine in the docker container, this way the container can manage host's docker service from inside the docker container. (**use with caution**)

```shell
docker run -v /var/run/docker.sock:/var/run/docker.sock docker
```

Use [`dive`](https://github.com/wagoodman/dive) with docker in docker so you don't need to install `dive` on your machine

```shell
docker run --rm -it -v /var/run/docker.sock:/var/run/docker.sock -v $(which docker):/bin/docker wagoodman/dive <image to inspect>
```

## Multistage build

```
# Stage 1. Build cache and ensure the code passes all tests
FROM node:18 as builder
COPY ["package.json", "package-lock.json", "/usr/src/"]

WORKDIR /usr/src
RUN npm install --omit=dev

COPY [".", "/usr/src/"]
RUN npm install --omit=dev

RUN npm run test

# Stage 2. Build production image and use the cached layers
FROM node:18-alpine3.15

COPY ["package.json", "package-lock.json", "/usr/src/"]

WORKDIR /usr/src
RUN npm install --omit=dev

COPY --from=builder ["/usr/src/index.js", "/usr/src/"]
EXPOSE 3000

ENTRYPOINT ["node", "index.js"]
```

## Other tips

Add a `.dockerignore` so your image doesn't contain more files than the ones actually needed.
