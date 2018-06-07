This introduction continues where the previous one left off and covers some additional topics related to running containers.

### Mounting Volumes ###
By default, when you run a docker container, its file system is completely isolated from the host file system and only
contains the files that were added to the image. Through the use of "volume mounts", we can add files or directories
from the host to the container at run time to accomplish various goals.

Reasons to do use volume mounts include:
1. Save/persist files produced by the container beyond the container's life.
2. Parameterize the container with additional configuration files at run time.
3. Add security sensitive files such as SSL certificates to a public image.

To mount a volume into a container use the `-v` flag: the format is `-f <host_path>:<conatiner_path>`. For example:

```
docker run -it -v /root/test:/data ubuntu bash
```

Will create and run a docker conainer from the ubuntu image with the `/root/test` directory on the host mounted at `/data`
from within the container. Note that changes made within the container persist onto the host!

```
Exercise. Create a container with a volume mount to some directory on the host and add create a text file with some
basic text from within the container. Verify that the text file exists on the host.
```


### Container Networking and Exposing Ports ###
In addition to an isolated file system, containers enjoy an isolated network stack on a special Docker network. In
general, the kind of special network used for containers depends on how Docker is set up. When multiple docker computers
are involved (called a "Docker swarm cluster") networks can be built to connect containers across hosts. For this
basic introduction however, we will restrict our discussion to the case of a single computer running Docker.

In the case of a single Docker host, containers are added to a "bridge" network. All the details of Docker bridge networking
are beyond the scope of this course, but here are some key facts:

* All containers get assigned an IP on the bridge network.
* By default, this IP address is in the 172.17.0.0 subnet and ports on it are available on the host only.
* A port on a container's IP address can be "mapped" to a (potentially different) port on the host's IP address to make it available externally.

We can determine which IP address a container was assigned by using the `docker inspect <container_id>` command. For example:
```
$ docker inspect 7443bd1063b9 | grep IPAddress
            "SecondaryIPAddresses": null,
            "IPAddress": "172.17.0.2",
                    "IPAddress": "172.17.0.2",
```
indicates that container with ID `7443bd1063b9` has IP address `172.17.0.2`.

#### Mapping Ports ####
Container ports can be mapped to the host using the `-p` flag to the `docker run` command: the format is `-p <host_port>:<container_port>`.

For example, the following statement would map port 9000 on the host to port 6379 within a redis container:
```
$ docker run -p 9000:6379 redis
. . .
The server is now ready to accept connections on port 6379
```
Note that we now have two ways to interact with redis: assuming the Redis container had IP address `172.17.0.2` and that the VM had a public IP address of `129.114.17.201`, from the VM, we can make requests to 172.17.0.2:6379 or (from outside) 129.114.17.201:9000.

```
Exercise. Nginx is a popular http proxy and has an official image available on the Docker Hub. By default, nginx listens for requests on port 80. Create an nginx container and map its port 80 to an external port on your VM's IP.
Exercise. Use curl to check that you can a) interact with nginx on port 80 using it's container IP and b) interact with nginx on the mapped port using your VM's IP.

### Building the Jupyter Image ###
Let's build a Docker image that contains a (nearly) identical software environment to that running in our VM:

```
Exercise. Copy or download the Dockerfil source code for the Jupyter image from the following URL: https://github.com/TACC/CSC2018Institute/blob/master/docker/Dockerfile
Exercise. Build a new docker image on your VM using this source.
```


### Running the Jupyter Container
We need to map the 8887 port so that Jupyter is available from outside the VM. We might also want to mount the current working directory on the host to a data directory to save our files after the container is destroyed.

```
docker run -d -it -p 8887:8887 -v $(pwd):/data tacc/csc_jupyter
```



