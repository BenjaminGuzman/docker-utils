# docker-utils
Docker util commands

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
