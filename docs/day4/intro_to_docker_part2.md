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



