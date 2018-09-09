## Working with containers

Get an interactive bash prompt in an already running container

```bash
docker exec -i -t [containerId] /bin/bash
```

Get an interactive bash prompt in an already running container as a specific user

```
docker exec -i -t --user=[username] [containerId] /bin/bash
```

Exit a container (from command prompt in container):

```
exit
```

List all running containers

```
docker ps
```

Stop a running container:

```
docker kill [containerId]
```

Remove a stopped container:

```
docker rm [containerId]
```

Remove all stopped containers:

```bash
docker container prune
```



## Working with images

List all stored images:

```
docker images
```

Delete a stored image:

```
docker rmi [imageId]
```

Remove all unused images

```
docker image prune
```



## Getting help

Get help on docker:

```
docker --help
```

Get help on a specific command:

```
docker [command] --help
```

