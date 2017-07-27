# Singularity @ TACC

We are continuously rolling out new super computers at TACC, and their operating systems and software stacks usually vary different to support specialized hardware or secure environments. To support tools that are community built and often not compatible with our system libraries, we have begun rolling out [Singularity](http://singularity.lbl.gov/index.html).

Singularity was designed to run on shared systems from the start. At no point can a user become root inside a container (as they would in a VM or docker) when they cannot on the main system itself. All permissions are preserved so no shared resources are affected. By this design an image cannot be edited on a TACC system.

What you can do with Singularity is build your own custom environment on your personal system, and then ship THAT image to TACC. Assuming you have sudo on your own system, you can `apt-get` or `yum install` any software you need. This stays inside the container. You then bring that container to TACC and it gets executed by your user account. You can also pull Docker images to an HPC system without being root too.

Apart from being secure, Singularity tries to pass system resources directly into the containers. This means shared filesystems will be available if they are usually available to you as a user, GPUs, and network interconnects that include infiniband.

## Installing Singularity

Lets begin by building Singularity on your Jetstream instances, so you can build a custom Singularity container.

```
sudo apt-get install libtool
git clone https://github.com/singularityware/singularity.git
cd singularity/
git checkout 2.3.1
./autogen.sh 
./configure --prefix=/usr/local --sysconfdir=/etc
make
sudo make install
```

## Creating a singularity image

Create an empty 1024 MB image and format it as an ext3 filesystem.

```
singularity create -s 1024 myimage.img
```

## Import an operating system

We are running ubuntu on our Jetstream instances, so lets import a centos system.

```
singularity import myimage.img docker://centos:latest
```

## Enter the container

We can enter the container and launch a shell with

```
singularity shell myimage.img
```

You can then interact with the container as you would a normal CLI. Exit your container, and lets do a couple sanity checks.

#### From Jetstream instance

```
[jetstream]$ whoami
[jetstream]$ cat /etc/*release
```
#### From Singularity shell

```
[jetstream]$ singularity shell myimage.img
[singularity]$ whoami
[singularity]$ cat /etc/*release
```

## Running Commands

You can also run commands in the container without entering it.

```
[jetstream]$ singularity exec myimage.img cat /etc/*release
```

While this location is inside your container, your $HOME directory gets passed into the container automatically.

```
[jetstream]$ echo "OUTSIDE CONTAINER" > ~/test.txt
[jetstream]$ singularity exec myimage.img cat $HOME/test.txt
```

## Modifying a container

By default, you cannot modify an image

```
[jetstream]$ singularity shell myimage.img
[singularity]$ touch /test.file
```

You can only modify a container with sudo, and specifying that you want to mount the container as writeable.

```
[jetstream]$ sudo singularity shell --writeable myimage.img
[singularity]$ touch /test.file
[singularity]$ exit
[jetstream]$ singularity exec myimage.img ls /
```

## Passing in filesystems

In normal versions of Singularity, you can specify which folders get mounted in a container when it is run. At TACC, we specifically limit that to our shared filesystems as a security precaution. This means you will need to always add

- /corral
- /home1
- /work
- /scratch

folders in your containers if you wish to interact with them like you did your `$HOME` folder.

## Community containers

We already know about pulling images like centos directly from docker, but there are whole communities like biocontainers for bioinformatics and singularity hub to deliver software. Lets try pulling qiime from biocontainers since it has a complicated install process.

```
[jetstream]$ singularity pull --size 2048 docker://quay.io/biocontainers/qiime:1.9.1--np112py27_1
[jetstream]$ singularity exec qiime-1.9.1--np112py27_1.img pick_de_novo_otus.py -h
```

Just make sure to

```
[jetstream]$ sudo singularity exec --writable qiime.img mkdir /{work,scratch,home1,corral}
```

if you want to run this at TACC. Lets also transfer this image to Stampede2

```
[jetstream]$ scp qiime-1.9.1--np112py27_1.img username@stampede2.tacc.utexas.edu
```

## Running containers at TACC

Singularity 2.3.1 is installed on Stampede2, but it can only be run from compute nodes.

```
[jetstream]$ ssh username@stampede2.tacc.utexas.edu
[stampede2]$ module load tacc-singularity
[stampede2]$ singularity -h
```

You should recieve a warning message because you are currently on a login node.

Login nodes are the nodes that all users land on when they ssh to a TACC system. These are designed to

- Handle network connections
- Stage data
- Schedule jobs

To ensure that all users have a responsive experience at TACC, we disabled Singularity on login nodes. This is because your container spawns many processes and utilizes loop devices when mounting the image. Luckily, TACC makes interactive compute sessions fairly accessible.

```
[jetstream]$ idev -m 30
[compute]$ singularity -h
```

You should now be able to run/exec/shell your the qiime container you transferred to Stampede2!

```
[compute]$ singularity exec qiime-1.9.1--np112py27_1.img pick_de_novo_otus.py -h
[compute]$ singularity shell qiime-1.9.1--np112py27_1.img
[singularity]$ ls /work
[singularity]$ ls /scratch
```

# Sinularity Bootstrap

```
Bootstrap: docker
From: tensorflow/tensorflow:latest

%runscript
 
# What happens when you "singularity run"
    exec /usr/bin/python "$@"

%post

# Extra installation commands
    echo "Post install stuffs!"

%files

# The files you want to add to the container
/home/vanessa/Desktop/analysis.py /tmp/analysis.py
relative_path.py /tmp/analysis2.py

%environment

# Environment variables
TOPSECRET pancakes
HELLO WORLD

%labels

# Container tags
AUTHOR username
```
