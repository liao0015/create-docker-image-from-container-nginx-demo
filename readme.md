# How to create a docker image from a container

## setup a simple nginx project

```shell
# create a base container
$ docker create --name nginx_base -p 80:80 nginx:alpine

# run the container
$ docker start nginx_base

# visit localhost or 192.168.99.100 to see nginx default web page

# modify the running container
$ cd project/repo
$ docker cp .\index.html nginx_base:/usr/share/nginx/html/index.html

# refresh the browser to see the changes in effect
```

## create an image from the container

- docker commit doesn't require a running container

```shell
# purpose: save this container as an image so that we can build other containers based on this base container

# create a new image from the running container
$ docker commit nginx_base
# tag the new image
$ docker tag <image id> my_nginx_base
# or in one command with version
$ docker commit nginx_base my_nginx_base:1.0.0
```

## test with the new image

```shell
# make sure you stop and romove the running container
# as it is still using port 80, which will prevent you launching a new image at the same port

# run a new container based on the new image
$ docker run --name hi_nginx -d -p 80:80 my_nginx_base
```

## more ways to commit your custom docker images

`docker commit <container name> <new image name>`

```shell
# set authorship

# inspect a docker image
$ docker inspect my_nginx_base
# or specifically
$ docker inspect my_nginx_base | grep Author

# commit with author
$ docker commit --author lude@skywalker.com nginx_base my_nginx_base:1.0.0

# to remove the image
$ docker rmi my_nginx_base:1.0.0
```

```shell
# create commit messages

# commit with a message
$ docker commit --message "init image" nginx_base my_nginx_base:1.0.0
# check the message in docker history
$ docker history my_nginx_base:1.0.0
```

```shell
# commit without pause
# Important: When you use the commit command, the container will be paused.  You may not have noticed the pause for this simple example

# you can use 'pause=false'
$ docker commit --pause=false nginx_base my_nginx_base:1.0.0

# However: If you don't pause the container, you run the risk of corrupting your data. For example, if the container is in the midst of a write operation, the data being written could be corrupted or come out incomplete. That is why, by default, the container gets paused before the image is created.
```

## change configuration

The `-c` or `–change` flag allows you to set the configuration of the image. You can change any of the following settings of the image during the commit process:

- CMD
- ENTRYPOINT
- ENV
- EXPOSE
- LABEL
- ONBUILD
- USER
- VOLUME
- WORKDIR

In addition: Nginx's original docker file contains the following settings:

- `CMD ["nginx", "-g", "daemon off;"]`
- `ENV NGINX_VERSION 1.15.3`
- `EXPOSE 80`
- TIP: Nginx allows us to pass the `-T` command line argument that will dump its configuration to standard out.

```shell
# create a new image with changed nginx configuration
$ docker commit --change='CMD ["nginx", "-T"]' nginx_base nginx-config-dump
# run the new container
$ docker run --name my_nginx_dumper -p 80:80 nginx-config-dump

# unfortunately, the CMD part doesn't work for powershell...to be continued...
```